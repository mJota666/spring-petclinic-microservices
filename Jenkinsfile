pipeline {
    agent any
    environment {
        SERVICE = "none"
    }
    stages {
        stage('Detect Changes') {
            steps {
                script {
                    // Run git diff and capture raw output.
                    def rawChangedFiles = bat(script: "git diff --name-only HEAD~1", returnStdout: true).trim()
                    echo "Raw Changed Files: [${rawChangedFiles}]"
                    
                    // Split the raw output into lines.
                    def lines = rawChangedFiles.split(/[\r\n]+/)
                    // Find the first line that starts with "spring-petclinic-"
                    def serviceLine = lines.find { it.trim().startsWith("spring-petclinic-") }
                    def normalizedChangedFiles = serviceLine != null ? serviceLine.trim() : ""
                    echo "Normalized Changed Files: [${normalizedChangedFiles}]"
                    
                    // Convert to lowercase for case-insensitive matching.
                    def norm = normalizedChangedFiles.toLowerCase()
                    echo "Lowercase Normalized: [${norm}]"
                    
                    // Check which service folder is modified.
                    if (norm.contains("jenkinsfile")) {
                        echo "Jenkinsfile was updated. Skipping microservice build."
                        env.SERVICE = "none"
                    } else if (norm.contains("spring-petclinic-vets-service")) {
                        echo "Detected vets service change."
                        env.SERVICE = "spring-petclinic-vets-service"
                    } else if (norm.contains("spring-petclinic-customers-service")) {
                        echo "Detected customers service change."
                        env.SERVICE = "spring-petclinic-customers-service"
                    } else if (norm.contains("spring-petclinic-genai-service")) {
                        echo "Detected genai service change."
                        env.SERVICE = "spring-petclinic-genai-service"
                    } else if (norm.contains("spring-petclinic-visits-service")) {
                        echo "Detected visits service change."
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
                    // Strip the "spring-petclinic-" prefix to derive the simple agent label.
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
