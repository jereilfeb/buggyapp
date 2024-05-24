pipeline {
    agent any

    environment {
      SONAR_TOKEN = withCredentials('SONAR_TOKEN')
    }
    tools { 
        maven 'maven'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/jereilfeb/buggyapp.git'
            }
        }
        stage('CompileandRunSonarAnalysis') {
            steps {	
                sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=buggy-app_buggy-app -Dsonar.organization=buggy-app -Dsonar.host.url=https://sonarcloud.io'
            } 
        }
    }
}