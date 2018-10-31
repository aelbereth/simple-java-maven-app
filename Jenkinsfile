pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test') { 
            steps {
                sh 'mvn test' 
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml' 
                }
            }
        }
	stage('SonarQube analysis') {
	  steps {
	    withSonarQubeEnv('SonarQube') {
	      // requires SonarQube Scanner for Maven 3.2+
	      sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.2:sonar'
	    }
	  }
	}
	stage('Deliver') { 
            steps {
                sh './jenkins/scripts/deliver.sh' 
            }
        }
    }
}
