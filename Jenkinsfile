pipeline {
    agent any

    stages {

        stage('Check Environment') {
            steps {
                sh 'hostname'
                sh 'pwd'
                sh 'docker --version'
                sh 'aws --version'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t jenkins-ci-demo:latest .'
            }
        }

        stage('Test') {
            steps {
                sh '''
                    docker run -d --name jenkins-ci-test -p 8082:80 jenkins-ci-demo:latest

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
                    945488515668.dkr.ecr.ap-south-1.amazonaws.com

                    docker tag jenkins-ci-demo:latest \
                    945488515668.dkr.ecr.ap-south-1.amazonaws.com/jenkins-ci-demo:latest

                    docker push \
                    945488515668.dkr.ecr.ap-south-1.amazonaws.com/jenkins-ci-demo:latest
                '''
            }
        }

        stage('SSH Test') {
            steps {
                sshagent(['jenkins-production-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no \
                        ubuntu@15.252.144.112 \
                        "hostname && whoami"
                    '''
                }
            }
        }
    }
}
