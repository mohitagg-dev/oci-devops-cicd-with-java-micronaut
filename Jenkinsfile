pipeline {
    agent {
        kubernetes {
            serviceAccount "jenkins-admin"
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
    - name: jenkins-agent
      image: "jenkins/jnlp-agent:latest" # Use an appropriate Jenkins JNLP agent image
      resources:
        limits:
          memory: "512Mi"
          cpu: "500m"
        requests:
          memory: "256Mi"
          cpu: "250m"
      env:
        # Crucial: This tells the Jenkins agent to connect to the DinD sidecar
        - name: DOCKER_HOST
          value: "tcp://localhost:2375" # Connection endpoint for the Docker daemon
      # Add a command to ensure the daemon is running before the agent connects
      command: ["sh", "-c", "sleep 5 && java -jar /usr/share/jenkins/jenkins-agent.jar"] # Wait for DinD to start
    - name: docker
      image: docker:24.0-dind
      securityContext:
        privileged: true
      resources:
        limits:
          memory: "512Mi"
          cpu: "500m"
        requests:
          memory: "256Mi"
          cpu: "250m"
      env:
        # Expose the daemon via TCP on port 2375
        - name: DOCKER_TLS_CERTDIR
          value: "" # Disable TLS for local communication within the pod
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
                container('docker') {
                echo 'Create Container Image..'
                sh 'docker --version'
                echo "Current workspace is ${env.WORKSPACE}"
                script {
                    def currentDir = pwd()
                    echo "Current working directory is: ${currentDir}"
                }
                sh "ls -la /var/run/"
                sh 'docker info' // Test connection
                sh 'dockerd-entrypoint.sh'
                sh 'docker status'
                //sh 'docker build -t demo-repo/java_micronaut_sample:${BUILD_NUMBER} .'
                }
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
