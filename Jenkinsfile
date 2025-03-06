pipeline {
    agent any
    environment {
        SERVICE = ""
    }
    stages {
        stage('Detect Changes') {
            steps {
                script {
                    def changedFiles = bat(script: "git diff --name-only HEAD~1", returnStdout: true).trim()
                    echo "Changed Files: ${changedFiles}"

                    if (changedFiles.contains("vets-service/")) {
                        env.SERVICE = "vets-service"
                    } else if (changedFiles.contains("customers-service/")) {
                        env.SERVICE = "customers-service"
                    } else if (changedFiles.contains("genai-service/")) {
                        env.SERVICE = "genai-service"
                    } else if (changedFiles.contains("visits-service/")) {
                        env.SERVICE = "visits-service"
                    } else {
                        error "No relevant service was modified."
                    }

                    echo "Service to build: ${env.SERVICE}"
                }
            }
        }

        stage('Test') {
            agent { label "${env.SERVICE}-agent" }
            steps {
                script {
                    echo "Running tests for ${env.SERVICE}"
                    bat "cd ${env.SERVICE} && mvn test"
                }
            }
            post {
                always {
                    junit "${env.SERVICE}/target/surefire-reports/*.xml"
                }
            }
        }

        stage('Build') {
            agent { label "${env.SERVICE}-agent" }
            steps {
                script {
                    echo "Building ${env.SERVICE}"
                    bat "cd ${env.SERVICE} && mvn package"
                }
            }
        }

        stage('Deploy') {
            agent { label "${env.SERVICE}-agent" }
            steps {
                script {
                    echo "Deploying ${env.SERVICE}..."
                    bat "docker build -t myrepo/${env.SERVICE}:latest ${env.SERVICE}"
                    bat "docker push myrepo/${env.SERVICE}:latest"
                }
            }
        }
    }
}
