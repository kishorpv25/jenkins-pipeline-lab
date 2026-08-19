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

        stage('Run Container') {
            steps {
                sh 'docker run -d --name jenkins-ci-demo-container -p 8081:80 jenkins-ci-demo'
            }
        }

    }
}
