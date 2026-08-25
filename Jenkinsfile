pipeline {
    agent any

    stages {

        stage('Check Environment') {
            steps {
                sh 'hostname'
                sh 'pwd'
                sh 'docker --version'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t jenkins-ci-demo .'
            }
        }

        stage('Test') {
            steps {
                sh '''
                    docker run -d --name jenkins-ci-test -p 8082:80 jenkins-ci-demo
                    sleep 3
                    curl -f http://localhost:8082
                    docker stop jenkins-ci-test
                    docker rm jenkins-ci-test
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                sh 'docker stop jenkins-ci-demo-container || true'
                sh 'docker rm jenkins-ci-demo-container || true'
                sh 'docker run -d --name jenkins-ci-demo-container -p 8081:80 jenkins-ci-demo'
            }
        }
    }
}
