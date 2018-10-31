pipeline {
#    agent {
#        docker {
#            image 'maven:3-alpine'
#            args '-v /root/.m2:/root/.m2'
#        }
#    }
    stages {
	stage ("initialize") {
	 steps {
		sh '''
		echo "PATH = ${PATH}"
		echo "M2_HOME = ${M2_HOME}"
		'''
	  }
	}
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
