pipeline {
    agent any
    environment {
        SERVICE = "none"  // ✅ Set default to prevent "null-agent" issue
    }
    stages {
        stage('Detect Changes') {
            steps {
                script {
                    def changedFiles = bat(script: "git diff --name-only HEAD~1", returnStdout: true).trim()
                    echo "Changed Files: ${changedFiles}"

                    if (changedFiles.contains("Jenkinsfile")) {
                        echo "Jenkinsfile was updated. Skipping microservice build."
                        env.SERVICE = "none"
                    } else if (changedFiles.contains("spring-petclinic-vets-service/")) {
                        env.SERVICE = "spring-petclinic-vets-service"
                    } else if (changedFiles.contains("spring-petclinic-customers-service/")) {
                        env.SERVICE = "spring-petclinic-customers-service"
                    } else if (changedFiles.contains("spring-petclinic-genai-service/")) {
                        env.SERVICE = "spring-petclinic-genai-service"
                    } else if (changedFiles.contains("spring-petclinic-visits-service/")) {
                        env.SERVICE = "spring-petclinic-visits-service"
                    } else {
                        echo "No relevant service was modified. Skipping pipeline."
                        env.SERVICE = "none"
                    }

                    echo "Service to build: ${env.SERVICE}"
                }
            }
        }

        stage('Test') {
            when {
                expression { env.SERVICE != "none" }
            }
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
            when {
                expression { env.SERVICE != "none" }
            }
            agent { label "${env.SERVICE}-agent" }
            steps {
                script {
                    echo "Building ${env.SERVICE}"
                    bat "cd ${env.SERVICE} && mvn package"
                }
            }
        }

        stage('Deploy') {
            when {
                expression { env.SERVICE != "none" }
            }
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
