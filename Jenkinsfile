def DIND_IMAGE = 'docker:20.10.16-dind'
def AGENT_IMAGE = 'jenkins/inbound-agent:4.11.2-4' // Example standard Jenkins agent image

podTemplate(
    label: 'dind-build-agent',
    cloud: 'kubernetes', // Ensure your Jenkins Kubernetes cloud is named correctly
    containers: [
        containerTemplate(
            name: 'jnlp',
            image: AGENT_IMAGE,
            args: '${computer.jnlpUrl} ${computer.secret}',
            resourceRequestCpu: '100m',
            resourceRequestMemory: '256Mi'
        )
    ],
    // The dind container is added as a sidecar 'service'
    services: [
        containerTemplate(
            name: 'docker', // Name the container 'docker' for easy reference
            image: DIND_IMAGE,
            args: '--storage-driver=overlay2', // Optional: specify storage driver
            resourceRequestCpu: '500m',
            resourceRequestMemory: '1024Mi',
            // Dind requires specific security context and privileges
            privileged: true,
            tty: true,
            command: 'cat' // Keep the container running
        )
    ],
    // Ensure the pod has a service account with K8s permissions if needed for K8s interaction
    serviceAccount: 'jenkins-admin' 
) {
    node('dind-build-agent') {
        stage('Build Docker Image') {
            // Use the 'docker' container environment
            container('docker') {
                sh 'docker info'
                sh 'docker build -t my-image:latest .'
                // You will need credentials configured in Jenkins for pushing to a registry
                // sh "docker login -u myuser -p mypassword myregistry.com" 
                // sh 'docker push my-image:latest'
            }
        }
    }
}
