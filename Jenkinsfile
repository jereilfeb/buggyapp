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
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) 
                        sh """
                        mvn clean verify sonar:sonar \
                        "-Dsonar.projectKey=buggy-app-test" \
                        "-Dsonar.organization=buggy-app-test" \
                        2"-Dsonar.host.url=https://sonarcloud.io" \
                        "-Dsonar.login=${SONAR_TOKEN}"
                        """
            } 
        
            }
            stage('RunSCANAnalysisUsingSynk') {
            steps{
                withCredentials([sting(credentialsId: 'SYNK_TOKEN', variable: 'SYNK_TOKEN')]) {
                    sh 'mvn snyk:test -fn'
                }
        }


        }
    }
}

