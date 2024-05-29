pipeline {
    agent any

    tools { 
        maven 'maven'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/jereilfeb/buggyapp.git'
            }
        }

        stage('Compile and Run Sonar Analysis') {
            steps {    
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR')]) {
                    sh 'mvn clean verify sonar:sonar "-Dsonar.projectKey=buggy-app-test" "-Dsonar.organization=buggy-app-test" "-Dsonar.host.url=https://sonarcloud.io" "-Dsonar.token=$SONAR"'
                }
            }
        }

        stage('Run SCAN Analysis Using Synk') {
            steps {
                withCredentials([string(credentialsId: 'SNYK_API_KEY', variable: 'SNYK_TOKEN')]) {
                    sh 'mvn snyk:test -fn'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    app = docker.build("buggy")
                }
            }
        }

        stage('Tag Docker Image') {
            steps {
                script {
                    // Tag Docker image
                    sh "docker tag buggy 975050199901.dkr.ecr.us-east-1.amazonaws.com/buggy:latest"
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    // Push Docker image to ECR
                    withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-credentials', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh "docker push https://975050199901.dkr.ecr.us-east-1.amazonaws.com/buggy:latest"
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up Docker image
            sh 'docker rmi -f buggy:latest'
        }
    }
}
