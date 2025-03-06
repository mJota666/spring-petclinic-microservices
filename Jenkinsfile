pipeline {
    agent any
    parameters {
        choice(name: "VERSION", choices:['1.0', '2.0', '3.0'], description: "Deploy-version")
        string(name: "VERSION_2", defaultValue: "1.0,2.0,3.0", description: "Deploy-version-2")
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
                // bat "echo Deploy version: ${params.VERSION}"
                def versions = params.VERSION_2.split(",")
                for (version in versions) {
                    bat "echo deploy version ${version}"
                }
            }
        }
    }
}
