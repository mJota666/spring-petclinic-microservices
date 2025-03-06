pipeline {
    agent any
    parameters {
        choice(name: "VERSION", choices:['1.0', '2.0', '3.0'], description: "Deploy-version")
    }
    stages {
        stage('Build') {
            steps {
                bat "echo Build project . . ."
            }
        }
        stage('Deploy') {
            steps {
                bat "echo Deploy project . . ."
                bat "Deploy version: ${params.version}"
            }
        }
    }
}
