// Global variable to store detected service.
def detectedService = "none"

pipeline {
    agent any
    stages {
        stage('Detect Changes') {
            steps {
                script {
                    // Check if this is a PR or a feature branch.
                    if (env.CHANGE_ID != null || (env.BRANCH_NAME != null && env.BRANCH_NAME != "main")) {
                        echo "New branch or PR detected. Running full pipeline for all services."
                        detectedService = "all"
                    } else {
                        // Run git diff to capture changed files.
                        def rawChangedFiles = bat(script: "git diff --name-only HEAD~1", returnStdout: true).trim()
                        echo "Raw Changed Files: [${rawChangedFiles}]"
                        
                        // Split output into lines.
                        def lines = rawChangedFiles.split(/[\r\n]+/)
                        // Find the first line that starts with "spring-petclinic-"
                        def serviceLine = lines.find { it.trim().startsWith("spring-petclinic-") }
                        def normalizedChangedFiles = serviceLine != null ? serviceLine.trim() : ""
                        echo "Normalized Changed Files: [${normalizedChangedFiles}]"
                        
                        // Convert to lowercase.
                        def norm = normalizedChangedFiles.toLowerCase()
                        echo "Lowercase Normalized: [${norm}]"
                        
                        // Diff-based detection.
                        if (norm.contains("jenkinsfile")) {
                            echo "Only Jenkinsfile changed. Running full pipeline for all services."
                            detectedService = "all"
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
                            echo "No specific service detected. Running full pipeline for all services."
                            detectedService = "all"
                        }
                    }
                    echo "Detected service: ${detectedService}"
                    env.DETECTED_SERVICE = detectedService
                }
            }
        }
        
        stage('Test') {
            when {
                expression { env.DETECTED_SERVICE != "none" && env.DETECTED_SERVICE != "" }
            }
            steps {
                script {
                    if (env.DETECTED_SERVICE == "all") {
                        def services = [
                            "spring-petclinic-vets-service",
                            "spring-petclinic-customers-service",
                            "spring-petclinic-genai-service",
                            "spring-petclinic-visits-service"
                        ]
                        services.each { svc ->
                            def simpleName = svc.replace("spring-petclinic-", "")
                            def agentLabel = "${simpleName}-agent"
                            echo "Testing service ${svc} on agent: ${agentLabel}"
                            node(agentLabel) {
                                checkout scm
                                bat "cd ${svc} && mvnw.cmd test"
                                junit "${svc}/target/surefire-reports/*.xml"
                            }
                        }
                    } else {
                        def svc = env.DETECTED_SERVICE
                        def simpleName = svc.replace("spring-petclinic-", "")
                        def agentLabel = "${simpleName}-agent"
                        echo "Testing service ${svc} on agent: ${agentLabel}"
                        node(agentLabel) {
                            checkout scm
                            bat "cd ${svc} && mvnw.cmd test"
                            junit "${svc}/target/surefire-reports/*.xml"
                        }
                    }
                }
            }
        }
        
        stage('Build') {
            when {
                expression { env.DETECTED_SERVICE != "none" && env.DETECTED_SERVICE != "" }
            }
            steps {
                script {
                    if (env.DETECTED_SERVICE == "all") {
                        def services = [
                            "spring-petclinic-vets-service",
                            "spring-petclinic-customers-service",
                            "spring-petclinic-genai-service",
                            "spring-petclinic-visits-service"
                        ]
                        services.each { svc ->
                            def simpleName = svc.replace("spring-petclinic-", "")
                            def agentLabel = "${simpleName}-agent"
                            echo "Building service ${svc} on agent: ${agentLabel}"
                            node(agentLabel) {
                                checkout scm
                                bat "cd ${svc} && mvnw.cmd clean package -DskipTests"
                            }
                        }
                    } else {
                        def svc = env.DETECTED_SERVICE
                        def simpleName = svc.replace("spring-petclinic-", "")
                        def agentLabel = "${simpleName}-agent"
                        echo "Building service ${svc} on agent: ${agentLabel}"
                        node(agentLabel) {
                            checkout scm
                            bat "cd ${svc} && mvnw.cmd clean package -DskipTests"
                        }
                    }
                }
            }
        }
        
        stage('Deploy') {
            when {
                expression { env.DETECTED_SERVICE != "none" && env.DETECTED_SERVICE != "" }
            }
            steps {
                script {
                    if (env.DETECTED_SERVICE == "all") {
                        def services = [
                            "spring-petclinic-vets-service",
                            "spring-petclinic-customers-service",
                            "spring-petclinic-genai-service",
                            "spring-petclinic-visits-service"
                        ]
                        services.each { svc ->
                            def simpleName = svc.replace("spring-petclinic-", "")
                            def agentLabel = "${simpleName}-agent"
                            echo "Deploying service ${svc} on agent: ${agentLabel}"
                            node(agentLabel) {
                                checkout scm
                                bat "docker stop ${svc} || echo 'No container to stop'"
                                bat "docker rm ${svc} || echo 'No container to remove'"
                                bat "docker build -t myrepo/${svc}:latest ${svc}"
                                bat "docker run -d --name ${svc} -p 8080:8080 myrepo/${svc}:latest"
                            }
                        }
                    } else {
                        def svc = env.DETECTED_SERVICE
                        def simpleName = svc.replace("spring-petclinic-", "")
                        def agentLabel = "${simpleName}-agent"
                        echo "Deploying service ${svc} on agent: ${agentLabel}"
                        node(agentLabel) {
                            checkout scm
                            bat "docker stop ${svc} || echo 'No container to stop'"
                            bat "docker rm ${svc} || echo 'No container to remove'"
                            bat "docker build -t myrepo/${svc}:latest ${svc}"
                            bat "docker run -d --name ${svc} -p 8080:8080 myrepo/${svc}:latest"
                        }
                    }
                }
            }
        }
    }
}
