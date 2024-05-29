def checkout() {
    stage('Checkout') {
        steps {
            git branch: 'main', url: 'https://github.com/jereilfeb/buggyapp.git'
        }
    }
}

def compileAndRunSonarAnalysis() {
    stage('Compile and Run Sonar Analysis') {
        steps {    
            withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR')]) {
                sh 'mvn clean verify sonar:sonar "-Dsonar.projectKey=buggy-app-test" "-Dsonar.organization=buggy-app-test" "-Dsonar.host.url=https://sonarcloud.io" "-Dsonar.token=$SONAR"'
            }
        }
    }
}

def runScanAnalysisUsingSynk() {
    stage('Run SCAN Analysis Using Synk') {
        steps {
            withCredentials([string(credentialsId: 'SNYK_API_KEY', variable: 'SNYK_TOKEN')]) {
                sh 'mvn snyk:test -fn'
            }
        }
    }
}

def buildDockerImage() {
    stage('Build Docker Image') {
        steps {
            script {
                // Build Docker image
                app = docker.build("buggy")
            }
        }
    }
}

def tagDockerImage() {
    stage('Tag Docker Image') {
        steps {
            script {
                // Tag Docker image
                sh "docker tag buggy 975050199901.dkr.ecr.us-east-1.amazonaws.com/buggy:latest"
            }
        }
    }
}

def pushDockerImageToECR() {
    stage('Push Docker Image to ECR') {
        steps {
            withCredentials([usernamePassword(credentialsId: 'aws-credentials', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                // Login to AWS ECR securely
                sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 975050199901.dkr.ecr.us-east-1.amazonaws.com"

                // Push Docker image to ECR
                sh "docker push 975050199901.dkr.ecr.us-east-1.amazonaws.com/buggy:latest"
            }
        }
    }
}

pipeline {
    agent any

    tools { 
        maven 'maven'
    }

    stages {
        checkout()
        compileAndRunSonarAnalysis()
        runScanAnalysisUsingSynk()
        buildDockerImage()
        tagDockerImage()
        pushDockerImageToECR()
    }

    post {
        always {
            // Clean up Docker image
            sh 'docker rmi -f buggy:latest'
        }
    }
}
