pipeline {
    agent any

    parameters {
        string(
            name: 'DEPLOY_TAG',
            defaultValue: '',
            description: 'Leave empty for normal deployment. Enter an existing ECR tag for rollback.'
        )
    }

    environment {
        ECR_REGISTRY = "945488515668.dkr.ecr.ap-south-1.amazonaws.com"
        IMAGE_NAME = "jenkins-ci-demo"
    }

    stages {

        stage('Determine Image Tag') {
            steps {
                script {
                    if (params.DEPLOY_TAG?.trim()) {

                        env.IMAGE_TAG = params.DEPLOY_TAG.trim()

                        echo "Rollback deployment selected"
                        echo "IMAGE_TAG = ${env.IMAGE_TAG}"

                    } else {

                        env.IMAGE_TAG = env.GIT_COMMIT[0..7]

                        echo "Normal deployment selected"
                        echo "IMAGE_TAG = ${env.IMAGE_TAG}"
                    }
                }
            }
        }

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
            when {
                expression {
                    !params.DEPLOY_TAG?.trim()
                }
            }

            steps {
                sh '''
                    docker build \
                    -t ${IMAGE_NAME}:${IMAGE_TAG} .
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
            when {
                expression {
                    !params.DEPLOY_TAG?.trim()
                }
            }

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
                        ubuntu@13.126.138.195\
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

                echo "Successfully deployed ${IMAGE_NAME}:${IMAGE_TAG} to Production"
            }
        }
    }
}
