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
                    sh 'docker run -d --name flask-test simple-flask-app'
                    sh 'sleep 5'
                    sh 'docker exec flask-test curl -f http://localhost:5000'
                    sh 'docker stop flask-test'
                    sh 'docker rm flask-test'
                }
            }
        }
        stage('Push to ECR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-access-key', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    script {
                        env.AWS_ACCESS_KEY_ID = "${AWS_ACCESS_KEY_ID}"
                        env.AWS_SECRET_ACCESS_KEY = "${AWS_SECRET_ACCESS_KEY}"
                        def region = 'us-east-1' // e.g., us-east-1
                        def repo = '833371734412.dkr.ecr.us-east-1.amazonaws.com/simple-flask-app' // your ECR repo URI

                        // Authenticate Docker to ECR
                        sh '''
                          aws configure set aws_access_key_id ${env.AWS_ACCESS_KEY_ID}
                          aws configure set aws_secret_access_key ${env.AWS_SECRET_ACCESS_KEY}
                          aws configure set default.region ${region}
                          aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${repo}
                        '''

                        // Tag and push the image
                        sh "docker tag simple-flask-app:latest ${repo}:latest"
                        sh "docker push ${repo}:latest"
                    }
                }
            }
        }
    }
}