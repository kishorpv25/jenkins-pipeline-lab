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
    }
}
