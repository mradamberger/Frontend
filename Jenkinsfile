def imageName="192.168.44.44:8082/docker-local/frontend"
def dockerRegistry="https://192.168.44.44:8082"
def registryCredentials="artifactory"
def dockerTag=""



pipeline {
    agent {
  label 'agent'
}

 environment {
        scannerHome = tool 'SonarQube'
    }

    stages {
        stage('Clone ansible roles') {
            steps {
		checkout scm
            }
        }
        stage('Show cloned repository') {
            steps {
                sh 'ls'
            }
        }
        
        stage('Run Tests') {
            steps {
                sh 'pip3 install -r requirements.txt'
                sh 'python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml'
            }
        }
        
        stage('Sonarqube analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Build application image') {
            steps {
                script {
                  // Prepare basic image for application
                dockerTag = "1.0"
                applicationImage = docker.build("$imageName:$dockerTag",".")
                }
            }
        }   
        
        
stage ('Pushing image to Artifactory') {
            steps {
                script {
                    docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                    applicationImage.push()
                    applicationImage.push('latest')
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

