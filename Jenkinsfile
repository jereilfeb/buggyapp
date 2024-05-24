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
        stage('CompileandRunSonarAnalysis') {
            steps {	
                sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=buggy-app_buggy-app -Dsonar.organization=buggy-app -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=ec8201cf5a8cf2909debb1c24d1680279e2fdb60'
            } 
        }
    }
}