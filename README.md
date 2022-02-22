# jenkins-declarative
Jenkins declarative pipeline for demo Java application build and provisioning
All test are optional and could be disabled/enabled at runtime, the same for docker image build/push

## Prerequisites
1. Jenkins server installed with CloudFormation template (based on Ubuntu 20.04 AMI, please set right AMI for your region)
2. Initialized Jenkins and installed plugins: Docker Pipeline/Docker plugin/Git plugin/Gradle/JUnit/OWASP/Pipeline/SonarQube
3. Java demo application copied locally and defined as Git source - https://github.com/jenkins-docs/simple-java-maven-app
4. Jenkinsfile copied from this repo to demo application folder
5. Multi-branch Jenkins pipeline set from local Git repository
6. SonarQube server started in the same network and configured in Jenkins Settings
7. AWS ECR regitry created and set as an parameter in Jenkinsfile
8. AWS account ID and region probided as parameters in Jenkinsfile
