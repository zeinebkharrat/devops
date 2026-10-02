pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'SonarQube'
    }

    stages {
        stage('GIT') {
            steps {
                checkout scm
                sh 'chmod +x backend/mvnw'
            }
        }

        stage('Build') {
            steps {
                dir('backend') {
                    sh './mvnw -B clean compile -DskipTests'
                }
            }
        }

        stage('Tests') {
            steps {
                dir('backend') {
                    sh './mvnw -B test'
                }
            }
            post {
                always {
                    junit testResults: 'backend/target/surefire-reports/*.xml', allowEmptyResults: true
                }
            }
        }

        stage('SonarQube') {
            steps {
                dir('backend') {
                    withSonarQubeEnv(env.SONARQUBE_SERVER) {
                        sh './mvnw -B sonar:sonar'
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Package') {
            steps {
                dir('backend') {
                    sh './mvnw -B package -DskipTests'
                }
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'backend/target/*.jar', fingerprint: true
        }
        always {
            cleanWs()
        }
    }
}
