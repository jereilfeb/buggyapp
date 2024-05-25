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
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SECRET_VARIABLE')])
                sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=git-buggy_git-buggy -Dsonar.organization=git-buggy -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=SECRET_VARIABLE'
            } 
        }
    }
}
