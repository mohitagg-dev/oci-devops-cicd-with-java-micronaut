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
            steps {
                echo 'Compiling the code..'
                sh './gradlew -Dhttp.proxyHost=10.115.208.19 -Dhttp.proxyPort=80 -Dhttps.proxyHost=10.115.208.19 -Dhttps.proxyPort=80 clean build -x test --debug'
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
                script {
                    def dockerHome = tool 'docker'
                    env.PATH = "${dockerHome}/bin:${env.PATH}"
                    /*sh 'getent group docker || groupadd docker'
                    sh 'usermod -aG docker jenkins'
                    echo 'Jenkins user added to the docker group. Permissions updated.'*/
                }
                echo 'Create Container Image..'
                sh 'docker build --pull --rm -t java_micronaut_sample .'
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
