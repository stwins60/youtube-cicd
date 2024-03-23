pipeline {
    agent any
    tools {
        nodejs 'node21'
    }
    environment {
        DOCKER_CREDENTIALS = credentials('d4506f04-b98c-47db-95ce-018ceac27ba6')
        SCANNER_HOME = tool 'sonar-scanner'
        IMAGE_NAME = 'idrisniyi94/youtube-cicd'
    }
    stages {
        stage('Clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/stwins60/youtube-cicd.git'
            }
        }
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh "${tool 'sonar-scanner'}/bin/sonar-scanner -Dsonar.projectKey=youtube-cicd -Dsonar.sources=."
                }
            }
        }
    }
}