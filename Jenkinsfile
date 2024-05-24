pipeline {
  agent any
  tools { 
        maven 'maven'
    }
   stages{
    stage('CompileandRunSonarAnalysis') {
            steps {	
		sh '
			}mvn clean verify sonar:sonar -Dsonar.projectKey=asgbuggy_asgbuggyapp -Dsonar.organization=asgbuggy -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=460d68124e7d57e8eabbd140917ba549980fdf71'
        } 
  }
}
