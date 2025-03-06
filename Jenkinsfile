pipeline {
    agent any
    environment {
        SERVICE = "none"
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

        stage('Test, Build & Deploy') {
            when {
                expression { env.SERVICE != "none" && env.SERVICE != "" }
            }
            steps {
                script {
                    // Strip the "spring-petclinic-" prefix from the service name.
                    def simpleName = env.SERVICE.replace("spring-petclinic-", "")
                    def agentLabel = "${simpleName}-agent"
                    echo "Using agent: ${agentLabel}"
                    
                    node(agentLabel) {
                        echo "Running tests for ${env.SERVICE}"
                        bat "cd ${env.SERVICE} && mvn test"
                        junit "${env.SERVICE}/target/surefire-reports/*.xml"
                        
                        echo "Building ${env.SERVICE}"
                        bat "cd ${env.SERVICE} && mvn package"
                        
                        echo "Deploying ${env.SERVICE}..."
                        bat "docker build -t myrepo/${env.SERVICE}:latest ${env.SERVICE}"
                        bat "docker push myrepo/${env.SERVICE}:latest"
                    }
                }
            }
        }
    }
}
