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
                        withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                            sh './mvnw -B clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.token="$SONAR_TOKEN"'
                        }
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
                archiveArtifacts artifacts: 'backend/target/*.jar', fingerprint: true
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
