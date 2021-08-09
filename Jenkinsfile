def runflag = 'y'
pipeline {
    agent any
    parameters {
        booleanParam(name: 'Static_Check')
        booleanParam(name: 'QA')
        booleanParam(name: 'Unit_Test')
        string(name: 'Success_Email')
        string(name: 'Failure_Email')
    }
    stages {
        stage('Git Pull') {
            steps {
                echo "Pulling git repository"
                //git branch: 'main', url: '<ENTER GITHUB REPOSITORY URL>'
            }
        }
        stage('Is the run required?') {
            steps {
                script {
                    def today = new Date()
                    def date = today.format("yyyy-MM-dd")
                    def year = today.format("yyyy")

                    def response = httpRequest "https://calendarific.com/api/v2/holidays?&api_key=d20d05ccb411d9ce3b56b654971e17a29b0aa1ed&country=IN&year=${year}"
                    writeFile file: 'response.json', text: response.content.trim()
                    def jsonfile = readJSON file: "${WORKSPACE}/response.json"

                    holidaydates = jsonfile.response.holidays.date.iso.toString()
                    if(holidaydates.contains(date)) {
                    	echo 'Today is a holiday.'
                        runflag = 'n'
                    }
                }
            }
        }
        stage('Build') {
            when{
                expression {
                    runflag == 'y';
                }             
            }
            steps {
                script{
                    def buildfile = readJSON file: "${WORKSPACE}/Build.json"
                    buildfile.each { key, value ->
                        dir ('builds'){
                            writeFile file: "$value.Name" + '.txt', text: "$value.Content", encoding: 'UTF-8'
                        }
                    }
                    zip dir: 'builds', overwrite: true, zipFile: 'builds.zip'
                } 
            }
        }
        stage(' '){
            parallel{
                stage('Quality'){
                    stages{
                        stage('Static check') {
                            when{
                                allOf{
                                expression{return params.Static_Check}
                                expression{runflag == 'y'}
                            	}
                            }
                            steps {
                                script{
                                    def branchName = "${STAGE_NAME}"
                                    def globstr = branchName.split(' ')[0]
                                    unzip dir: "${STAGE_NAME}", glob: '*' + "$globstr" + '*', zipFile: 'builds.zip'
                                }
                                catchError(stageResult: 'FAILURE') {
                                    // some block
                                }
                            }
                        }
                        stage('QA') {
                            when{
                            	allOf{
                                expression{return params.QA}
                                expression{runflag == 'y'}
                                }
                            }
                            steps {
                                script{
                                    def branchName = "${STAGE_NAME}"
                                    def globstr = branchName.split(' ')[0]
                                    unzip dir: "${STAGE_NAME}", glob: '*' + "$globstr" + '*', zipFile: 'builds.zip'
                                }
                            }
                        }
                    }
                }
                stage('Unit test') {
                    when{
                        allOf{
                        expression{return params.Unit_Test}
                        expression{runflag == 'y'}
                        }
                    }
                    steps {
                        script{
                            def branchName = "${STAGE_NAME}"
                            def globstr = branchName.split(' ')[0]
                            unzip dir: "${STAGE_NAME}", glob: '*' + "$globstr" + '*', zipFile: 'builds.zip'
                        }
                    }
                }

            }
         }
        stage('Summary') {
            steps {
                script{
                    if(params.Static_Check){
                        def files = findFiles(glob: 'Static*/**')
                        echo 'Static Check executed. Copied - ' + """${files[0].name}"""
                    }
                    if(params.QA){
                        def files = findFiles(glob: 'QA*/**')
                        echo 'QA executed. Copied - ' + """${files[0].name}"""
                    }
                    if(params.Unit_Test){
                        def files = findFiles(glob: 'Unit*/**')
                        echo 'Unit Test executed. Copied - ' + """${files[0].name}"""
                    }
                }
            }
        }
    }
    post {
         success {
            echo "Sending mail to ${params.Success_Email}"
            emailext body: 'The job ran successfully', subject: 'Job Notification - Success', to: "${params.Success_Email}"
         }
         failure {
            echo "Sending mail to ${params.Failure_Email}"
            emailext body: 'The job failed', subject: 'Job Notification - Failure', to: "${params.Failure_Email}"
         }
     }
}
