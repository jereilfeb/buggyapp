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
                    withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
                        app = docker.build("buggy")
                    }
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    // Tag Docker image for ECR
                    docker.withRegistry('https://975050199901.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:AWS_CREDENTIALS') {
                        app.tag("latest", "975050199901.dkr.ecr.us-east-1.amazonaws.com/buggy:latest")
                    }
                    // Push Docker image to ECR
                    docker.withRegistry('', 'ecr:us-east-1:AWS_CREDENTIALS') {
                        app.push("latest")
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