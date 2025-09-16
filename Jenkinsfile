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
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-access-key']
                ]) {
                    script {
                        def region = 'us-east-1' // e.g., us-east-1
                        def repo = '833371734412.dkr.ecr.us-east-1.amazonaws.com/simple-flask-app' // your ECR repo URI

                        sh "aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${repo}"
                        sh "docker tag simple-flask-app:latest ${repo}:latest"
                        sh "docker push ${repo}:latest"
                    }
                }
            }
        }
        stage('Deploy to Fargate') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-access-key']
                ]) {
                    script {
                        def region = 'us-east-1'
                        def image = '833371734412.dkr.ecr.us-east-1.amazonaws.com/simple-flask-app:latest'
                        def cluster = 'lesley-simple-flask-app'
                        def service = 'lesley-simple-flask-app-task-service-wmm0jno5'
                        def taskdef_file = 'taskdef.json'
                        // Replace image in taskdef.json
                        sh "sed -i 's|REPLACE_IMAGE|${image}|g' ${taskdef_file}"
                        // Register new task definition and capture the output
                        def taskDefArn = sh(
                            script: "aws ecs register-task-definition --cli-input-json file://${taskdef_file} --region ${region} | jq -r .taskDefinition.taskDefinitionArn",
                            returnStdout: true
                        ).trim()
                        // Update ECS service to use new task definition
                        sh "aws ecs update-service --cluster ${cluster} --service ${service} --task-definition ${taskDefArn} --region ${region}"
                    }
                }
            }
        }
    }
}