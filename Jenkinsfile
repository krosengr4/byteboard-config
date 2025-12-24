pipeline {
    agent any

    parameters {
        string(name: 'BUILD_NUMBER', defaultValue: 'latest', description: 'Docker image tag/build number to deploy')
    }

    environment {
        PI_HOST = 'rosenpi.local'
        PI_USER = 'krosengren'
        DOCKER_IMAGE = "krosengr4/byteboard-ui:${params.BUILD_NUMBER}"
    }

    stages {
        stage('Deploy to K3s') {
            steps {
                echo 'Deploying byteboard-ui to K3s...'
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'rosenpi-ssh',
                    keyFileVariable: 'SSH_KEY'
                )]) {
                    script {
                        def imageTag = "${DOCKER_IMAGE}"
                        sh """
                            # Create target directory
                            ssh -i \$SSH_KEY -o StrictHostKeyChecking=no \${PI_USER}@\${PI_HOST} 'mkdir -p ~/byteboard-ui-k8s'

                            # Copy files
                            scp -i \$SSH_KEY -o StrictHostKeyChecking=no -r k8s \${PI_USER}@\${PI_HOST}:~/byteboard-ui-k8s/
                            scp -i \$SSH_KEY -o StrictHostKeyChecking=no .env \${PI_USER}@\${PI_HOST}:~/byteboard-ui-k8s/

                            # Deploy
                            ssh -i \$SSH_KEY -o StrictHostKeyChecking=no \${PI_USER}@\${PI_HOST} << 'ENDSSH'
                                cd ~/byteboard-ui-k8s

                                if [ ! -d "k8s" ]; then
                                    echo "ERROR: k8s directory not found!"
                                    exit 1
                                fi

                                export \$(cat .env | xargs)
                                export DOCKER_IMAGE="${imageTag}"

                                for file in k8s/*.yaml; do
                                    echo "Applying: \$file (Image: ${imageTag})"
                                    envsubst < "\$file" | kubectl apply -f -
                                done

                                kubectl rollout status deployment/byteboard-ui --timeout=60s
                                kubectl get pods -l app=byteboard-ui
ENDSSH
                        """
                    }
                }
            }
        }
    }

    post {
        success { echo 'Deployment successful!' }
        failure { echo 'Deployment failed!' }
    }
}
