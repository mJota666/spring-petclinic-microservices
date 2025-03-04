pipeline {
    agent any  // Run on any available agent

    stages {
        stage('Environment Check') {
            steps {
                echo "Checking system information..."
                sh "uname -a"  // Check OS
                sh "java -version"  // Check Java version
                sh "mvn -version"  // Check Maven installation
            }
        }

        stage('Check Maven') {
            steps {
                sh "mvn -version"
            }
        }

        stage('Checkout') {
            steps {
                echo "Cloning repository..."
                checkout scm
                sh "ls -la"  // List files to confirm successful checkout
            }
        }

        stage('Test') {
            steps {
                echo "Running tests..."
                sh "chmod +x mvnw"  // Ensure mvnw is executable
                sh "./mvnw test || true"  // Run tests but do NOT fail pipeline
                sh "cat target/surefire-reports/*.xml || true"  // Print test reports
            }
        }

        stage('Build') {
            steps {
                echo "Building application..."
                sh "chmod +x mvnw"  // Ensure mvnw is executable
                sh "./mvnw package -DskipTests || true"  // Build but do NOT fail pipeline
            }
        }
    }
}
