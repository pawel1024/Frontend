def imageName="192.168.44.44:8082/docker-local/frontend"
def dockerRegistry="https://192.168.44.44:8082"
def registryCredentials="artifactory"
def dockerTag=""


pipeline {
    agent {label 'agent'}
    environment {scannerHome = tool 'SonarQube'}


    stages {
        stage('git_clone') {
            steps {
                checkout scm
            }
        }
        stage('install_requirements') {
            steps {
                sh 'pip3 install -r requirements.txt'
            }
        }
        stage('test_run') {
            steps {
                sh 'python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml'
            }
        }
        stage("build & SonarQube analysis") {
            steps {
              withSonarQubeEnv('SonarQube') {
                sh "${scannerHome}/bin/sonar-scanner"
              }
            }
          }
        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true 
              }
            }
        }
        stage('Build application image') {
            steps {
                script {
                    dockerTag="RC-${env.BUILD_ID}" // .${env.GIT_COMMIT.take(7)}"
                    applicationImage=docker.build("$imageName:$dockerTag")
                }
            }
        }
         stage('applicationImage to Artifactory') {
            steps {
                script {
                    docker.withRegistry("$dockerRegistry", "$registryCredentials"){
                    applicationImage.push()
                    applicationImage.push("latest")
                    }
                }
            }
        }
    }
    post {
        always {
            junit testResults: "test-results/*.xml"
            cleanWs()
        }
    }
}