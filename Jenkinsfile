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
//         stage('Checkout') {
//             steps {
//                 git branch: 'main', url: '
}