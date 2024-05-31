pipeline {
    agent any

    parameters {
        string(name: 'GIT_REPO_URL', defaultValue: 'https://github.com/jereilfeb/buggyapp.git', description: 'URL of the Git repository')
        string(name: 'SONAR_PROJECT_KEY', defaultValue: 'buggy-app-test', description: 'SonarCloud project key')
        string(name: 'SONAR_ORGANIZATION', defaultValue: 'buggy-app-test', description: 'SonarCloud organization key')
        string(name: 'DOCKER_IMAGE_NAME', defaultValue: 'buggy', description: 'Name of the Docker image')
        string(name: 'AWS_REGION', defaultValue: 'us-east-1', description: 'AWS Region')
        string(name: 'AWS_ECR_REPO_URL', defaultValue: '975050199901.dkr.ecr.us-east-1.amazonaws.com/buggy', description: 'AWS ECR repository URL')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: params.GIT_REPO_URL
            }
        }

        stage('Compile and Run Sonar Analysis') {
            steps {
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    sh "mvn clean verify sonar:sonar -Dsonar.projectKey=${params.SONAR_PROJECT_KEY} -Dsonar.organization=${params.SONAR_ORGANIZATION} -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${SONAR_TOKEN}"
                }
            }
        }

        stage('Run SCAN Analysis Using Synk') {
            steps {
                withCredentials([string(credentialsId: 'SNYK_API_KEY', variable: 'SNYK_API_KEY')]) {
                    sh 'mvn snyk:test -fn'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    docker.build(params.DOCKER_IMAGE_NAME)
                }
            }
        }

        stage('Tag Docker Image') {
            steps {
                script {
                    // Tag Docker image
                    docker.image(params.DOCKER_IMAGE_NAME).tag("latest")
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-credentials', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    // Login to AWS ECR securely
                    sh "aws ecr get-login-password --region ${params.AWS_REGION} | docker login --username AWS --password-stdin ${params.AWS_ECR_REPO_URL}"

                    // Push Docker image to ECR
                    sh "docker push ${params.AWS_ECR_REPO_URL}:latest"
                }
            }
        }

        stage('Kubernetes Deployment of Buggy Web Application') {
            steps {
                withKubeConfig([credentialsId: 'kubelogin']) {
                    sh('kubectl delete all --all -n devsecops')
                    sh('kubectl apply -f deployment.yaml --namespace=devsecops')
                }
            }
        }
    }

    post {
        always {
            // Clean up Docker image
            sh "docker rmi -f ${params.DOCKER_IMAGE_NAME}:latest"
        }
    }
}
