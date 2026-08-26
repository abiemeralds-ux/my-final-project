pipeline {

    agent any

    environment {
        GITHUB_REPO = 'https://github.com/abiemeralds-ux/my-final-project'
        EC2_HOST    = '18.209.221.194'
        EC2_USER    = 'ec2-user'
        APP_DIR     = '/home/ec2-user/my-final-project'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out repository...'

                checkout scm
            }
        }

        stage('Prepare EC2') {
            steps {
                echo 'Installing Docker and Docker Compose on EC2...'

                sshagent(['aws-ec2-ssh']) {

                    sh '''
                        ssh -o StrictHostKeyChecking=no \
                        ${EC2_USER}@${EC2_HOST} << 'REMOTE_COMMANDS'

                        set -e

                        echo "Updating packages..."
                        sudo apt-get update

                        echo "Installing required packages..."
                        sudo apt-get install -y \
                            ca-certificates \
                            curl \
                            git

                        # Install Docker if it is not already installed
                        if ! command -v docker >/dev/null 2>&1; then

                            echo "Installing Docker..."

                            sudo install -m 0755 -d /etc/apt/keyrings

                            sudo curl -fsSL \
                                https://download.docker.com/linux/ubuntu/gpg \
                                -o /etc/apt/keyrings/docker.asc

                            sudo chmod a+r /etc/apt/keyrings/docker.asc

                            echo \
                              "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
                              $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
                              sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

                            sudo apt-get update

                            sudo apt-get install -y \
                                docker-ce \
                                docker-ce-cli \
                                containerd.io \
                                docker-buildx-plugin \
                                docker-compose-plugin

                        else
                            echo "Docker already installed."
                        fi

                        # Make sure Docker service is running
                        sudo systemctl enable docker
                        sudo systemctl start docker

                        # Install Compose plugin if it isn't available
                        if ! sudo docker compose version >/dev/null 2>&1; then

                            echo "Installing Docker Compose plugin..."

                            sudo apt-get update
                            sudo apt-get install -y docker-compose-plugin

                        fi

                        echo "Docker version:"
                        docker --version || sudo docker --version

                        echo "Docker Compose version:"
                        sudo docker compose version

                        REMOTE_COMMANDS
                    '''
                }
            }
        }

        stage('Deploy Application') {
            steps {
                echo 'Cloning/updating GitHub repository on EC2...'

                sshagent(['aws-ec2-ssh']) {

                    sh '''
                        ssh -o StrictHostKeyChecking=no \
                        ${EC2_USER}@${EC2_HOST} << EOF

                        set -e

                        # Create application directory
                        sudo mkdir -p ${APP_DIR}

                        sudo chown -R ${EC2_USER}:${EC2_USER} ${APP_DIR}

                        # Clone repository if it doesn't exist
                        if [ ! -d "${APP_DIR}/.git" ]; then

                            echo "Cloning repository..."

                            git clone \
                                ${GITHUB_REPO} \
                                ${APP_DIR}

                        else

                            echo "Repository already exists. Updating..."

                            cd ${APP_DIR}

                            git fetch origin

                            git reset --hard origin/main

                        fi

                        cd ${APP_DIR}

                        echo "Project contents:"
                        ls -la

                        echo "Checking Compose file..."

                        if [ ! -f "docker-compose.yaml" ]; then
                            echo "ERROR: docker-compose.yaml not found!"
                            exit 1
                        fi

                        echo "Stopping previous deployment..."

                        sudo docker compose down || true

                        echo "Building and starting applications..."

                        sudo docker compose up -d --build

                        echo "Running containers..."

                        sudo docker compose ps

                        EOF
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Checking Portfolio...'

                sh '''
                    curl --fail --max-time 30 \
                    http://${EC2_HOST}/
                '''

                echo 'Checking Java application...'

                sh '''
                    curl --fail --max-time 30 \
                    http://${EC2_HOST}:8080/
                '''
            }
        }
    }

    post {

        success {
            echo '''
============================================
DEPLOYMENT SUCCESSFUL
============================================

Portfolio:
http://${EC2_HOST}/

Java Application:
http://${EC2_HOST}:8081/

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
