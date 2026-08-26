pipeline {
    agent any

    environment {
        EC2_HOST = '18.209.221.194'
        EC2_USER = 'ec2-user'
        APP_DIR  = '/home/ec2-user/my-final-project'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out repository...'
                checkout scm
            }
        }

        stage('Test Files') {
            steps {
                echo 'Checking project files...'

                sh '''
                    echo "Project directory:"
                    pwd

                    echo "Project files:"
                    ls -la

                    echo "Docker Compose file:"
                    test -f docker-compose.yaml
                '''
            }
        }

        stage('Deploy Application') {
            steps {
                echo 'Deploying application to EC2...'

                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'aws-ec2-ssh',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USERNAME'
                    )
                ]) {

                    sh '''
                        set -e

                        echo "Creating application directory on EC2..."

                        ssh -o StrictHostKeyChecking=no \
                            -i "$SSH_KEY" \
                            "$SSH_USERNAME@$EC2_HOST" \
                            "mkdir -p $APP_DIR"

                        echo "Copying project to EC2..."

                        scp -o StrictHostKeyChecking=no \
                            -i "$SSH_KEY" \
                            -r ./* \
                            "$SSH_USERNAME@$EC2_HOST:$APP_DIR/"

                        echo "Deploying containers..."

                        ssh -o StrictHostKeyChecking=no \
                            -i "$SSH_KEY" \
                            "$SSH_USERNAME@$EC2_HOST" \
                            "
                                cd $APP_DIR &&
                                docker compose down || true &&
                                docker compose up -d --build
                            "
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Checking deployed containers...'

                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'aws-ec2-ssh',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USERNAME'
                    )
                ]) {

                    sh '''
                        ssh -o StrictHostKeyChecking=no \
                            -i "$SSH_KEY" \
                            "$SSH_USERNAME@$EC2_HOST" \
                            "
                                echo 'Running containers:'
                                docker ps

                                echo ''
                                echo 'Docker Compose status:'
                                cd $APP_DIR
                                docker compose ps
                            "
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '''
============================================
DEPLOYMENT SUCCESSFUL
============================================
Application has been deployed to EC2.
============================================
'''
        }

        failure {
            echo '''
============================================
DEPLOYMENT FAILED
============================================
Check the Jenkins console output.
============================================
'''
        }
    }
}
