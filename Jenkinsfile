pipeline {
    agent none

    environment {
        SONARQUBE_URL = 'http://sonarqube:9000'
        SONARQUBE_CREDENTIALS = 'sonarqube-token'
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'maven:3.8.6-openjdk-11'
                    args '-v /root/.m2:/root/.m2'
                }
            }
            steps {
                script {
                    sh 'mvn clean compile'
                }
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'maven:3.8.6-openjdk-11'
                    args '-v /root/.m2:/root/.m2'
                }
            }
            steps {
                script {
                    sh 'mvn test'
                }
                junit 'target/surefire-reports/*.xml'
            }
        }

        stage('SonarQube Analysis') {
            agent {
                docker {
                    image 'sonarsource/sonar-scanner-cli:latest'
                }
            }
            steps {
                withSonarQubeEnv('SonarQube') {
                    script {
                        sh "sonar-scanner \
                            -Dsonar.projectKey=my-project \
                            -Dsonar.sources=src/main/java \
                            -Dsonar.tests=src/test/java \
                            -Dsonar.java.binaries=target/classes \
                            -Dsonar.host.url=${SONARQUBE_URL} \
                            -Dsonar.login=${SONARQUBE_CREDENTIALS}"
                    }
                }
            }
        }

        stage('Package') {
            agent {
                docker {
                    image 'maven:3.8.6-openjdk-11'
                    args '-v /root/.m2:/root/.m2'
                }
            }
            steps {
                script {
                    sh 'mvn package'
                }
                archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
