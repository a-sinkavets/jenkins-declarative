pipeline {
    agent none
    environment {
      awsAccount = ""
      awsRegion = ""
      registry = "AWS-ECR-IMAGE-REGISTRY"
    }
    parameters {
        booleanParam(
          defaultValue: false,
          description: 'Qwasp test',
          name: 'QWASP'
        )
        booleanParam(
          defaultValue: false,
          description: 'SonarQube test',
          name: 'SONAR'
        )
        booleanParam(
          defaultValue: true,
          description: 'Unit test',
          name: 'UNIT'
        )
        booleanParam(
          defaultValue: true,
          description: 'Image build and push to ECR',
          name: 'IMAGE'
        )
        booleanParam(
          defaultValue: true,
          description: 'Integration tests',
          name: 'INTEGRATION'
        )
    }
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build and test artifact') {
          agent {
              docker {
                  image 'maven:3.8.1-adoptopenjdk-11'
                  args '-v /root/.m2:/root/.m2'
              }
          }
          stages {
            stage('Build') {
              steps {
                  sh 'mvn -B -DskipTests clean package'
              }
            }
            stage('Unit test') {
              when {
                expression { return params.UNIT }
              }
              steps {
                sh 'mvn test'
              }
              post {
                always {
                  junit 'target/surefire-reports/*.xml'
                }
              }
            }
            stage ('OWASP Dependency-Check') {
              when {
                expression { return params.QWASP }
              }
              steps {
                dependencyCheck additionalArguments: '''
                  -o "./"
                  -s "./"
                  -f "ALL"
                  --prettyPrint''', odcInstallation: 'owasp'
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
              }
            }
            stage('SonarQube analysis') {
              when {
                expression { return params.SONAR }
              }
              steps {
                withSonarQubeEnv('SonarQube') {
                  sh "./gradlew sonarqube"
                }
              }
            }
            stage('Quality gate') {
              when {
                expression { return params.SONAR }
              }
              steps {
                waitForQualityGate abortPipeline: true
              }
            }
            stage('Deliver') {
              steps {
                sh './jenkins/scripts/deliver.sh'
              }
              post {
                always {
                  archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
              }
            }
          }
        }
        stage('Build/push image') {
          agent any
          when {
            expression { return params.IMAGE }
          }
          stages {
            stage('Copy artifact') {
              steps {
                sh 'cp ${WORKSPACE}/target/my-app-1.0-SNAPSHOT.jar my-app-1.0-SNAPSHOT.jar'
              }
            }
            stage('ECR Login') {
              steps {
		            sh 'aws ecr get-login-password --region eu-north-1 | docker login --username AWS --password-stdin ${awsAccount}.dkr.ecr.${awsRegion}.amazonaws.com'
              }
            }
            stage('Build Image') {
              steps {
                sh 'docker build -t jenkins-test:${BUILD_NUMBER} .'
              }
            }
            stage('Tag Image') {
              steps {
                sh 'docker tag jenkins-test:${BUILD_NUMBER} ${registry}:${BUILD_NUMBER}'
              }
            }
	          stage('Integration Test') {
              when {
                expression { return params.INTEGRATION }
              }
              steps {
                sh 'docker run jenkins-test:${BUILD_NUMBER} || exit 1'
              }
            }
            stage('Push Image') {
              steps {
                sh 'docker push ${registry}:${BUILD_NUMBER}'
              }
            }
          }
        }
    }
}
