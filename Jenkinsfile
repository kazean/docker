def repository="kazean/jenkins"
def deployHost="172.30.1.55"

pipeline {
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-jenkins')
    }
    agent any

    stages {
        stage('Pull Codes from Github'){
            steps{
                checkout scm
            }
        }
        stage('Build Codes by Gradle') {
            steps {
                sh """
                ./gradlew clean build
                """
            }
        }

        stage('Build Docker Image by Jib & Push to Docker Hub Repository') {
            steps {
                sh """
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    ./gradlew jib -Djib.to.image=${repository}:${currentBuild.number} -Djib.console='plain'
                """
            }
        }

        stage('Deploy to ubuntu'){
            steps{
                sshagent(credentials : ["deploy-key"]) {
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@${deployHost} \
                     'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin \
                      docker run -d -p 80:8080 -t ${repository}:${currentBuild.number};'"
                }
            }
        }
    }
}
