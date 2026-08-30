pipeline {

    agent {
        label 'jenkins-agent'
    }

    parameters {
        string(
            name: 'DEPLOY_TAG',
            defaultValue: '',
            description: 'Enter Docker image tag for rollback. Leave empty for normal deployment.'
        )
    }

    environment {
        AWS_REGION   = 'ap-south-1'
        ECR_REGISTRY = '945488515668.dkr.ecr.ap-south-1.amazonaws.com'
        IMAGE_NAME   = 'jenkins-ci-demo'
    }

    stages {

        stage('Determine Image Tag') {
            steps {
                script {

                    if (params.DEPLOY_TAG?.trim()) {

                        echo "Rollback deployment selected"

                        env.IMAGE_TAG = params.DEPLOY_TAG.trim()

                    } else {

                        echo "Normal deployment selected"

                        env.IMAGE_TAG = env.GIT_COMMIT[0..7]
                    }

                    echo "IMAGE_TAG = ${env.IMAGE_TAG}"
                }
            }
        }

        stage('Check Environment') {
            steps {
                sh '''
                    echo "=============================="
                    echo "Hostname:"
                    hostname

                    echo "=============================="
                    echo "User:"
                    whoami

                    echo "=============================="
                    echo "Directory:"
                    pwd

                    echo "=============================="
                    echo "Git Commit:"
                    echo "${GIT_COMMIT}"

                    echo "=============================="
                    echo "Image Tag:"
                    echo "${IMAGE_TAG}"

                    echo "=============================="
                    echo "Docker:"
                    docker --version

                    echo "=============================="
                    echo "AWS CLI:"
                    aws --version

                    echo "=============================="
                    echo "Ansible:"
                    ansible --version
                '''
            }
        }

        stage('Docker Build') {

            when {
                expression {
                    !params.DEPLOY_TAG?.trim()
                }
            }

            steps {
                sh '''
                    echo "Building Docker image..."

                    docker build \
                        -t ${IMAGE_NAME}:${IMAGE_TAG} .

                    echo "Docker image built successfully."
                '''
            }
        }

        stage('Test') {

            when {
                expression {
                    !params.DEPLOY_TAG?.trim()
                }
            }

            steps {
                sh '''
                    echo "Starting test container..."

                    docker run -d \
                        --name jenkins-ci-test \
                        -p 8082:80 \
                        ${IMAGE_NAME}:${IMAGE_TAG}

                    sleep 3

                    echo "Testing application..."

                    curl -f http://localhost:8082

                    echo "Application test successful."

                    docker stop jenkins-ci-test
                    docker rm jenkins-ci-test
                '''
            }
        }

        stage('Push to ECR') {

            when {
                expression {
                    !params.DEPLOY_TAG?.trim()
                }
            }

            steps {
                sh '''
                    echo "Logging into ECR..."

                    aws ecr get-login-password \
                        --region ${AWS_REGION} | \
                    docker login \
                        --username AWS \
                        --password-stdin ${ECR_REGISTRY}

                    echo "Tagging image..."

                    docker tag \
                        ${IMAGE_NAME}:${IMAGE_TAG} \
                        ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}

                    echo "Pushing image to ECR..."

                    docker push \
                        ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}

                    echo "Image pushed successfully."
                '''
            }
        }

        stage('Deploy to Production') {

            steps {

                sshagent(['jenkins-production-ssh']) {

                    sh '''
                        echo "Deploying ${IMAGE_NAME}:${IMAGE_TAG} to Production..."

                        ssh -o StrictHostKeyChecking=no ubuntu@13.126.138.195 << EOF

                        echo "Logging into ECR..."

                        aws ecr get-login-password \
                            --region ${AWS_REGION} | \
                        docker login \
                            --username AWS \
                            --password-stdin ${ECR_REGISTRY}

                        echo "Pulling image..."

                        docker pull \
                            ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}

                        echo "Stopping existing container..."

                        docker stop ${IMAGE_NAME} || true
                        docker rm ${IMAGE_NAME} || true

                        echo "Starting new container..."

                        docker run -d \
                            --name ${IMAGE_NAME} \
                            -p 80:80 \
                            ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}

                        sleep 3

                        echo "Checking container..."

                        docker ps --filter "name=${IMAGE_NAME}"

                        echo "Checking application..."

                        curl -f http://localhost

                        EOF
                    '''

                    echo "Successfully deployed ${IMAGE_NAME}:${IMAGE_TAG} to Production."
                }
            }
        }
    }
}
