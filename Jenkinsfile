pipeline {
    agent any
    stages {
        stage('Clone') {
            steps {
                checkout scm
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build('simple-flask-app')
                }
            }
        }
        stage('Test Docker Image') {
            steps {
                script {
                    sh 'docker run -d --name flask-test -p 6000:6000 simple-flask-app'
                    sh 'sleep 5'
                    sh 'curl -f http://localhost:6000'
                    sh 'docker stop flask-test'
                    sh 'docker rm flask-test'
                }
            }
        }
    }
}