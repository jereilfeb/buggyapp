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
                sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=git-buggy_git-buggy -Dsonar.organization=git_buggy -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=SONAR_TOKEN'
            } 
        }
    }
}