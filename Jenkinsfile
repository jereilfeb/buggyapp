pipeline {
  agent any
  tools { 
        maven 'maven'
    }
   stages{
    stage('CompileandRunSonarAnalysis') {
            steps {	
		sh 'mvn clean verify sonar:sonar "-Dsonar.projectKey=asgbuggyapp" "-Dsonar.organization=asgbuggy" "-Dsonar.host.url=https://sonarcloud.io" "-Dsonar.token=sonnar-scanner"'
			}
        } 
  }
}
