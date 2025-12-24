pipeline {
    agent any

    environment {
        PI_HOST = 'rosenpi.local'
        PI_USER = 'krosengren'
    }

    stages {
        stage('Deploy to K3s') {
            steps {
                echo 'Deploying byteboard-ui to K3s...'
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'rosenpi-ssh',
                    keyFileVariable: 'SSH_KEY'
                )]) {
                    sh '''
                        scp -i $SSH_KEY -o StrictHostKeyChecking=no -r k8s ${PI_USER}@${PI_HOST}:~/byteboard-ui-k8s
                        scp -i $SSH_KEY -o StrictHostKeyChecking=no .env ${PI_USER}@${PI_HOST}:~/byteboard-ui-k8s/

                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no ${PI_USER}@${PI_HOST} '
                            cd ~/byteboard-ui-k8s
                            export $(cat .env | xargs)

                            for file in k8s/*.yaml; do
                                envsubst < "$file" | kubectl apply -f -
                            done

                            kubectl rollout status deployment/byteboard-ui --timeout=60s
                            kubectl get pods -l app=byteboard-ui
                        '
                    '''
                }
            }
        }
    }

    post {
        success { echo 'Deployment successful!' }
        failure { echo 'Deployment failed!' }
    }
}
