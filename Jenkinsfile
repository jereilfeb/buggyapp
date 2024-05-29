pipeline {
    agent any

    parameters {
        string(name: 'GIT_REPO_URL', defaultValue: 'https://github.com/jereilfeb/buggyapp.git', description: 'URL of the Git repository')
        string(name: 'SONAR_PROJECT_KEY', defaultValue: 'buggy-app-test', description: 'SonarCloud project key')
        string(name: 'SONAR_ORGANIZATION', defaultValue: 'buggy-app-test', description: 'SonarCloud organization key')
        string(name: 'DOCKER_IMAGE_NAME', defaultValue: 'buggy', description: 'Name of the Docker image')
        string(name: 'AWS_REGION', defaultValue: 'us-east-1', description: 'AWS Region')
        string(name: 'AWS_ECR_REPO_URL', defaultValue: '975050199901.dkr.ecr.us-east-1.amazonaws.com/buggy', description: 'AWS ECR repository URL')
        string(name: 'ECS_CLUSTER_NAME', defaultValue: 'Test_Cluster', description: 'ECS Cluster Name')
        string(name: 'ECS_SERVICE_NAME', defaultValue: 'test', description: 'ECS Service Name')
        string(name: 'ECS_TASK_DEFINITION', defaultValue: 'task-dev01', description: 'ECS Task Definition Name')
        string(name: 'ECS_CONTAINER_NAME', defaultValue: 'test', description: 'ECS Container Name')
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
                    sh "docker tag ${params.DOCKER_IMAGE_NAME} ${params.AWS_ECR_REPO_URL}:latest"
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-credentials', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    // Login to AWS ECR securely
                    sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 975050199901.dkr.ecr.us-east-1.amazonaws.com"

                    // Push Docker image to ECR
                    sh "docker push ${params.AWS_ECR_REPO_URL}:latest"
                }
            }
        }

        stage('Deploy to ECS Fargate') {
            steps {
                withAWS(region: params.AWS_REGION, credentials: 'aws-credentials') {
                    ecsFargateDeploy(
                        cluster: params.ECS_CLUSTER_NAME,
                        service: params.ECS_SERVICE_NAME,
                        taskDefinition: params.ECS_TASK_DEFINITION,
                        containerName: params.ECS_CONTAINER_NAME,
                        image: "${params.AWS_ECR_REPO_URL}:latest"
                    )
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
