```groovy
pipeline {

    agent any

    environment {
        AWS_REGION = 'us-east-1'

        // Change this to your GitHub repository
        REPO_URL = 'https://github.com/YOUR-GITHUB-USERNAME/YOUR-REPOSITORY.git'

        // Terraform directory
        TF_DIR = 'terraform'

        // Ansible directory
        ANSIBLE_DIR = 'ansible'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'

                checkout scm
            }
        }

        stage('Verify Project') {
            steps {
                sh '''
                    echo "Project structure:"
                    ls -la

                    echo "Portfolio:"
                    ls -la portfolio

                    echo "Java application:"
                    ls -la java-app

                    echo "Terraform:"
                    ls -la terraform

                    echo "Ansible:"
                    ls -la ansible
                '''
            }
        }

        stage('Build Java Application') {
            steps {
                echo 'Building Java WAR application...'

                sh '''
                    docker run --rm \
                      -v "$PWD/java-app:/app" \
                      -w /app \
                      maven:3.8.8-eclipse-temurin-8 \
                      mvn clean package -DskipTests
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                echo 'Building Portfolio Docker image...'

                sh '''
                    docker build \
                      -t myportfolio:${BUILD_NUMBER} \
                      -t myportfolio:latest \
                      ./portfolio
                '''

                echo 'Building Java Docker image...'

                sh '''
                    docker build \
                      -t java-app:${BUILD_NUMBER} \
                      -t java-app:latest \
                      ./java-app
                '''

                echo 'Docker images:'

                sh '''
                    docker images | grep -E 'myportfolio|java-app'
                '''
            }
        }

        stage('Test Docker Images') {
            steps {
                echo 'Testing Docker images...'

                sh '''
                    docker run -d \
                      --name portfolio-test-${BUILD_NUMBER} \
                      -p 8181:80 \
                      myportfolio:${BUILD_NUMBER}
                '''

                sh '''
                    docker run -d \
                      --name java-test-${BUILD_NUMBER} \
                      -p 8180:8080 \
                      java-app:${BUILD_NUMBER}
                '''

                sleep 15

                sh '''
                    curl --fail http://localhost:8181
                '''

                sh '''
                    curl --fail http://localhost:8180
                '''
            }

            post {
                always {
                    sh '''
                        docker rm -f \
                          portfolio-test-${BUILD_NUMBER} \
                          java-test-${BUILD_NUMBER} \
                          2>/dev/null || true
                    '''
                }
            }
        }

        stage('Terraform Init') {
            steps {
                echo 'Initializing Terraform...'

                dir("${TF_DIR}") {
                    sh '''
                        terraform init
                    '''
                }
            }
        }

        stage('Terraform Validate') {
            steps {
                echo 'Validating Terraform configuration...'

                dir("${TF_DIR}") {
                    sh '''
                        terraform validate
                    '''
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                echo 'Creating Terraform plan...'

                dir("${TF_DIR}") {
                    sh '''
                        terraform plan \
                          -out=tfplan
                    '''
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                echo 'Provisioning AWS infrastructure...'

                dir("${TF_DIR}") {
                    sh '''
                        terraform apply \
                          -auto-approve \
                          tfplan
                    '''
                }
            }
        }

        stage('Get EC2 IP') {
            steps {
                script {
                    env.EC2_IP = sh(
                        script: """
                            cd ${TF_DIR}
                            terraform output -raw ec2_public_ip
                        """,
                        returnStdout: true
                    ).trim()

                    echo "EC2 Public IP: ${env.EC2_IP}"
                }
            }
        }

        stage('Ansible Configuration') {
            steps {
                echo 'Configuring EC2 server with Ansible...'

                dir("${ANSIBLE_DIR}") {
                    sh '''
                        ansible-playbook \
                          -i inventory.ini \
                          playbook.yml \
                          --extra-vars "ec2_ip=${EC2_IP}"
                    '''
                }
            }
        }

        stage('Deploy Containers') {
            steps {
                echo 'Deploying Portfolio and Java applications...'

                dir("${ANSIBLE_DIR}") {
                    sh '''
                        ansible-playbook \
                          -i inventory.ini \
                          deploy.yml \
                          --extra-vars "ec2_ip=${EC2_IP}"
                    '''
                }
            }
        }

        stage('Verification') {
            steps {
                echo 'Verifying applications...'

                sh '''
                    echo "Testing Portfolio..."
                    curl --fail --max-time 30 \
                      http://${EC2_IP}/
                '''

                sh '''
                    echo "Testing Java application..."
                    curl --fail --max-time 30 \
                      http://${EC2_IP}:8080/
                '''

                echo "======================================"
                echo "Deployment successful!"
                echo "Portfolio: http://${EC2_IP}/"
                echo "Java App:  http://${EC2_IP}:8080/"
                echo "======================================"
            }
        }
    }

    post {

        success {
            echo '======================================'
            echo 'DEVOPS PIPELINE COMPLETED SUCCESSFULLY'
            echo '======================================'
        }

        failure {
            echo '======================================'
            echo 'DEVOPS PIPELINE FAILED'
            echo 'Check the stage logs above.'
            echo '======================================'
        }

        always {
            echo 'Cleaning Jenkins workspace...'

            sh '''
                docker container prune -f || true
                docker image prune -f || true
            '''
        }
    }
}
```

