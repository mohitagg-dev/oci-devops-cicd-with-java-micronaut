pipeline {
    agent {
        kubernetes {
            // Define the Kubernetes Pod YAML inline
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: "jenkins-agent-build"
    serviceAccount: "jenkins-admin"
spec:
  containers:
    - name: docker
      image: docker:20.10.16-dind
      command: ['sleep']
      args: ['99d']
      securityContext:
        privileged: true
      # Mount the Docker socket if performing Docker-in-Docker builds
      volumeMounts:
        - name: dockersock
          mountPath: /var/run/docker.sock
  volumes:
    # Define a volume for the Docker socket
    - name: dockersock
      hostPath:
        path: /var/run/docker.sock
            """
        }
    }
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
