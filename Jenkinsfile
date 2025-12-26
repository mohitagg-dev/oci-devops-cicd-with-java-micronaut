pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Checkout code from Repository..'
                checkout changelog: true, poll: true, scm: [$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: []]
            }
        }
        stage('Build') {
            environment {
                HTTP_PROXY = "http://10.115.208.19:80"
                HTTPS_PROXY = "http://10.115.208.19:80"
                NO_PROXY = "127.0.0.1|localhost" 
            }
            steps {
                echo 'Compiling the code..'
                sh './gradlew clean build -x test'
            }
        }
        stage('Test') {
            steps {
                echo 'Running the Unit Test Cases..'
                sh './gradlew test'
            }
        }
        stage('Create Image') {
            steps {
                echo 'Create Container Image..'
            }
        }
        stage('Push Image') {
            steps {
                echo 'Push Container Image to Registry..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying the application to OKE....'
            }
        }
    }

    post {
        // Actions to run after the entire pipeline finishes
        always {
            echo 'Pipeline finished.'
            // Clean up workspace if needed
            // deleteDir() 
        }
        failure {
            echo 'Pipeline failed. Review console output.'
        }
    }
}
