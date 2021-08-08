# JenkinsAssignment

pre-requisites

Plugins:
Git
HTTP Request Plugin
Pipeline Utility Steps
Blue Ocean
Email Extension Plugin : Manage Jenkins -> Configure System -> Extended E-mail Notification - Configure email setup to be able to send emails.


Create a pipeline in jenkins.

In Pipeline section, select Pipeline script from SCM. Select SCM as Git and provide the repository URL
Select Credentials none
Specify the Branches to build as */main
Script Path = Jenkinsfile
Save

Click on Build Now to run.

