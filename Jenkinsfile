pipeline {
    agent any
    tools{
	maven 'Maven3'
    }
    stages {
        stage('Compilation and Analysis') {
            parallel {
                stage("Compilation") {
                    steps {
                        sh 'mvn -B -DskipTests clean package'
                    }
                }
                stage("Checkstyle") {
                    steps {
                        sh "mvn checkstyle:checkstyle"
                        step([$class: 'CheckStylePublisher',
                          canRunOnFailed: true,
                          defaultEncoding: '',
                          healthy: '100',
                          pattern: '**/target/checkstyle-result.xml',
                          unHealthy: '90',
                          useStableBuildAsReference: true
                        ])
                    }
                }
            }
        }
        stage('Testing') {
            steps {

                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage("Runing unit tests") {
            steps {
                try {
                    step([sh 'mvn test -Punit'])
                } catch(err) {
                    step([$class: 'JUnitResultArchiver', testResults:'**/target/surefire-reports/TEST-*UnitTest.xml'])
                    throw err
                }
                step([$class: 'JUnitResultArchiver', testResults:'**/target/surefire-reports/TEST-*UnitTest.xml'])
            }
        }
        stage("Runing integration tests") {
            steps {
                try {
                    step([sh 'mvn test -Pintegration'])
                } catch(err) {
                    step([$class: 'JUnitResultArchiver', testResults:'**/target/surefire-reports/TEST-'+ '*IntegrationTest.xml'])
                    throw err
                }
                step([$class: 'JUnitResultArchiver', testResults:'**/target/surefire-reports/TEST-'+ '*IntegrationTest.xml'])
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
        stage('M2Storage') {
            steps {
                sh './jenkins/scripts/deliver.sh'
            }
        }
        stage('Packaging'){
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

