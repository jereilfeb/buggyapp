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
                // Use the withCredentials block to securely access the secret text within this step
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR')]) {
                    sh 'mvn clean verify sonar:sonar "-Dsonar.projectKey=buggy-app-test" "-Dsonar.organization=buggy-app-test" "-Dsonar.host.url=https://sonarcloud.io" "-Dsonar.token=$SONAR"'
                }
            } 
        
            }
            stage('RunSCANAnalysisUsingSynk') {
            steps{
               withCredentials([string(credentialsId: 'SNYK_API_KEY', variable: 'SNYK_TOKEN')]) {
                    sh 'mvn snyk:test -fn'
                }
        }
     }
            stage('build') {
                steps {
                   withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
                      script {
                            app = docker.build("buggy")
                     }
                   }
               }
            }
            stage('Push') {
                steps {
                    docker.withDockerRegistry('https://975050199901.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:AWS_CREDENTIALS') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
    }
}

