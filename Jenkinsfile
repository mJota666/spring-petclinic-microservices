pipeline {
    agent any
    environment {
        SERVICE = "none"  // ✅ Default to "none" to avoid null issues
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

        // ✅ SKIP TEST STAGE IF NO SERVICE IS MODIFIED
        stage('Test') {
            when {
                allOf {
                    expression { env.SERVICE != "none" }
                    expression { env.SERVICE != "" }  // Ensure it's not empty
                }
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

        // ✅ SKIP BUILD STAGE IF NO SERVICE IS MODIFIED
        stage('Build') {
            when {
                allOf {
                    expression { env.SERVICE != "none" }
                    expression { env.SERVICE != "" }
                }
            }
            agent { label "${env.SERVICE}-agent" }
            steps {
                script {
                    echo "Building ${env.SERVICE}"
                    bat "cd ${env.SERVICE} && mvn package"
                }
            }
        }

        // ✅ SKIP DEPLOY STAGE IF NO SERVICE IS MODIFIED
        stage('Deploy') {
            when {
                allOf {
                    expression { env.SERVICE != "none" }
                    expression { env.SERVICE != "" }
                }
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
