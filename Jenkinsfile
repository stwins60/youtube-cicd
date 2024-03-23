pipeline {
    agent any
    parameters {
        choice(name: 'BUILD_TYPE', choices: ['Create', 'Delete'], description: 'Choose the type of build to run')
    }
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
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/stwins60/youtube-cicd.git']])
            }
        }
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh "${tool 'sonar-scanner'}/bin/sonar-scanner -Dsonar.projectKey=youtube-cicd -Dsonar.sources=."
                }
            }
        }
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        stage('Trivy File Scan') {
            steps {
                sh "trivy fs ."
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit --nvdApiKey 4bdf4acc-8eae-45c1-bfc4-844d549be812',
                odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Docker Login') {
            steps {
               sh "echo $DOCKER_CREDENTIALS_PSW | docker login -u $DOCKER_CREDENTIALS_USR --password-stdin"
               echo "Login Succeeded"
            }
        }
        stage('Docker Build') {
            steps {
                sh "docker build -t $IMAGE_NAME ."
            }
        }
        stage('Docker Push') {
            steps {
                sh "docker push $IMAGE_NAME"
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image $IMAGE_NAME"
            }
        }
        stage('Deploy') {
            when {
                expression {
                    params.BUILD_TYPE == 'Create'
                }
            }
            steps {
                echo 'Deploying the application'
                script {
                   sh "docker run -d -p 8077:3000 $IMAGE_NAME"
                }
            }
        }
        stage('Delete') {
            when {
                expression {
                    params.BUILD_TYPE == 'Delete'
                }
            }
            steps {
                echo 'Deleting the application'
                script {
                    sh "docker stop $(docker ps -a -q --filter ancestor=$IMAGE_NAME)"
                    sh "docker rm $(docker ps -a -q --filter ancestor=$IMAGE_NAME)"
                }
            }
        }
    }
    post {
        success {
            slackSend channel: '#alerts', color: 'good', message: "Build successful for ${env.JOB_NAME} - ${env.BUILD_NUMBER}"
        }
        failure {
            slackSend channel: '#alerts', color: 'danger', message: "Build failed for ${env.JOB_NAME} - ${env.BUILD_NUMBER}"
        }
    }
}