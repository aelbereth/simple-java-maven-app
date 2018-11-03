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
            stage("Tests and Deployment") {
                parallel 'Unit tests': {
                stage("Runing unit tests") {
                    try {
                        sh "./mvnw test -Punit"
                    } catch(err) {
                        step([$class: 'JUnitResultArchiver', testResults:
                          '**/target/surefire-reports/TEST-*UnitTest.xml'])
                        throw err
                    }
                   step([$class: 'JUnitResultArchiver', testResults:
                     '**/target/surefire-reports/TEST-*UnitTest.xml'])
                }
                    }, 'Integration tests': {
                    stage("Runing integration tests") {
                        try {
                            sh "./mvnw test -Pintegration"
                        } catch(err) {
                            step([$class: 'JUnitResultArchiver', testResults:
                              '**/target/surefire-reports/TEST-'
                                + '*IntegrationTest.xml'])
                            throw err
                        }
                        step([$class: 'JUnitResultArchiver', testResults:
                          '**/target/surefire-reports/TEST-'
                            + '*IntegrationTest.xml'])
                    }
            }
            stage('QA') {
              steps {
                withSonarQubeEnv('SonarQube') {
                  // requires SonarQube Scanner for Maven 3.2+
                  sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.2:sonar'
                }
              }
            }
            stage('M2Store') {
                steps {
                    sh './jenkins/scripts/deliver.sh'
                }
            }
            stage('Package'){
                when {
                    branch "master"
                }
                steps {
                    script {
                        def server = Artifactory.server('artifactory')
                        def rtMaven = Artifactory.newMavenBuild()
                        rtMaven.resolver server: server, releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot'
                        rtMaven.deployer server: server, releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local'
                        rtMaven.tool = 'Maven3'
                        def buildInfo = rtMaven.run pom: 'pom.xml', goals: 'install'
                        server.publishBuildInfo buildInfo
                    }
                }
            }
    }
    post {
        success {
            slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
        failure {
            slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
    }
}

