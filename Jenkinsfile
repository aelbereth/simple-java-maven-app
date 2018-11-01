pipeline {
    agent any
    tools{
	maven 'Maven3'
    }
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
	      sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.2:sonar -Dsonar.branch.name=development'
	    }
	  }
	}
	stage('Deliver') { 
            steps {
                sh './jenkins/scripts/deliver.sh' 
            }
        }
    }
    post {
        success {
            slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
            // Cucumber report plugin
            cucumber fileIncludePattern: '**/simple-java-maven-app/target/cucumber-report.json', sortingMethod: 'ALPHABETICAL'
            //publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: true, reportDir: '/home/reports', reportFiles: 'reports.html', reportName: 'Performance Test Report', reportTitles: ''])
        }
        failure {
            slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
            cucumber fileIncludePattern: '**/simple-java-maven-app/target/cucumber-report.json', sortingMethod: 'ALPHABETICAL'
            //publishHTML([allowMissing: true, alwaysLinkToLastBuild: false, keepAll: true, reportDir: '/home/tester/reports', reportFiles: 'reports.html', reportName: 'Performance Test Report', reportTitles: ''])
        }
    }
}
