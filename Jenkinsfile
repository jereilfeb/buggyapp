pipeline {
    agent any

    environment {
        // Declare the environment variable for SonarCloud token
        SONAR_TOKEN = credentials('SONAR_TOKEN')
    }

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
                // Use the environment variable SONAR_TOKEN
                sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=git-buggy_git-buggy -Dsonar.organization=git-buggy -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=1a3f9934eb3c8356ed969dff5103d52c571014d9'
            } 
        }
    }
}