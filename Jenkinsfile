pipeline {
    agent any

    environment {
        AWS_APP_HOST = '44.213.248.51'
        AZURE_APP_HOST = '52.150.20.9'
        AWS_USER = 'ubuntu'
        AZURE_USER = 'azureuser'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/neha436/multi-cloud-dr.git'
            }
        }

        stage('Deploy to AWS') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'multi-cloud-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                    sh """
                        scp -o StrictHostKeyChecking=no -i \$SSH_KEY index-aws.html ${AWS_USER}@${AWS_APP_HOST}:/tmp/index.html
                        ssh -o StrictHostKeyChecking=no -i \$SSH_KEY ${AWS_USER}@${AWS_APP_HOST} 'sudo cp /tmp/index.html /var/www/html/index.html && sudo systemctl restart nginx'
                    """
                }
            }
        }

        stage('Deploy to Azure') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'multi-cloud-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                    sh """
                        scp -o StrictHostKeyChecking=no -i \$SSH_KEY index-azure.html ${AZURE_USER}@${AZURE_APP_HOST}:/tmp/index.html
                        ssh -o StrictHostKeyChecking=no -i \$SSH_KEY ${AZURE_USER}@${AZURE_APP_HOST} 'sudo cp /tmp/index.html /var/www/html/index.html && sudo systemctl restart nginx'
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh """
                    echo "Verifying AWS deployment..."
                    curl -s http://${AWS_APP_HOST}:80 | grep -q "Welcome to AWS" && echo "AWS: OK" || echo "AWS: FAILED"

                    echo "Verifying Azure deployment..."
                    curl -s http://${AZURE_APP_HOST}:80 | grep -q "Welcome to Azure" && echo "Azure: OK" || echo "Azure: FAILED"
                """
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully!'
        }
        failure {
            echo 'Deployment failed. Check logs.'
        }
    }
}
