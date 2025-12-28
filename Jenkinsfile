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
        stage('Build Docker Image') {
            steps {
                // Run steps inside the 'docker' container
                container('docker') {
                    sh 'docker --version'
                    // Add steps here to build and push your application image
                    sh 'echo "Building application image..."'
                }
            }
        }
    }
}
