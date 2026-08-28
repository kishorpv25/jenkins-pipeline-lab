pipeline {
    agent any

    environment {
        IMAGE_TAG = "${GIT_COMMIT[0..7]}"
        ECR_REGISTRY = "945488515668.dkr.ecr.ap-south-1.amazonaws.com"
        IMAGE_NAME = "jenkins-ci-demo"
    }

    stages {

        stage('Check Environment') {
            steps {
                sh 'hostname'
                sh 'pwd'
                sh 'docker --version'
                sh 'aws --version'
                sh 'echo IMAGE_TAG=$IMAGE_TAG'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                    -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    docker run -d \
                    --name jenkins-ci-test \
                    -p 8082:80 \
                    ${IMAGE_NAME}:${IMAGE_TAG}

                    sleep 3

                    curl -f http://localhost:8082

                    docker stop jenkins-ci-test
                    docker rm jenkins-ci-test
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region ap-south-1 | \
                    docker login --username AWS --password-stdin \
                    ${ECR_REGISTRY}

                    docker tag \
                    ${IMAGE_NAME}:${IMAGE_TAG} \
                    ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}

                    docker push \
                    ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Deploy to Production') {
            steps {
                sshagent(['jenkins-production-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no \
                        ubuntu@13.201.166.172 \
                        "
                            aws ecr get-login-password --region ap-south-1 | \
                            docker login --username AWS --password-stdin \
                            ${ECR_REGISTRY}

                            docker pull \
                            ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}

                            docker stop jenkins-ci-demo || true
                            docker rm jenkins-ci-demo || true

                            docker run -d \
                            --name jenkins-ci-demo \
                            -p 80:80 \
                            ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}

                            sleep 3

                            echo 'Checking container status...'
                            docker ps --filter 'name=jenkins-ci-demo'

                            echo 'Checking application...'
                            curl -f http://localhost
                        "
                    '''
                }
            }
        }
    }
}
