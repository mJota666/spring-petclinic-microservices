def detectedService = "none"

pipeline {
    agent any
    stages {
        stage('Detect Changes') {
            steps {
                script {
                    // Run git diff and capture the raw output.
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
                    
                    // Use the global variable to capture the detected service.
                    if (norm.contains("jenkinsfile")) {
                        echo "Jenkinsfile was updated. Skipping microservice build."
                        detectedService = "none"
                    } else if (norm.contains("spring-petclinic-vets-service")) {
                        echo "Detected vets service change."
                        detectedService = "spring-petclinic-vets-service"
                    } else if (norm.contains("spring-petclinic-customers-service")) {
                        echo "Detected customers service change."
                        detectedService = "spring-petclinic-customers-service"
                    } else if (norm.contains("spring-petclinic-genai-service")) {
                        echo "Detected genai service change."
                        detectedService = "spring-petclinic-genai-service"
                    } else if (norm.contains("spring-petclinic-visits-service")) {
                        echo "Detected visits service change."
                        detectedService = "spring-petclinic-visits-service"
                    } else {
                        echo "No relevant service was modified. Skipping pipeline."
                        detectedService = "none"
                    }
                    
                    echo "Detected service: ${detectedService}"
                }
            }
        }

        stage('Test, Build & Deploy') {
            when {
                expression { detectedService != "none" && detectedService != "" }
            }
            steps {
                script {
                    // Derive the agent label by stripping "spring-petclinic-" prefix.
                    def simpleName = detectedService.replace("spring-petclinic-", "")
                    def agentLabel = "${simpleName}-agent"
                    echo "Using agent: ${agentLabel}"
                    
                    node(agentLabel) {
                        // Checkout the repository on the agent's workspace.
                        checkout scm
                        
                        echo "Running tests for ${detectedService}"
                        bat "cd ${detectedService} && mvn test"
                        junit "${detectedService}/target/surefire-reports/*.xml"
                        
                        echo "Building ${detectedService}"
                        bat "cd ${detectedService} && mvn package"
                        
                        echo "Deploying ${detectedService}..."
                        // bat "docker build -t myrepo/${detectedService}:latest ${detectedService}"
                        // bat "docker push myrepo/${detectedService}:latest"
                    }
                }
            }
        }
    }
}
