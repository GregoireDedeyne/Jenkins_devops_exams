pipeline {
    agent any

    environment {
        DOCKER_ID = 'gdedeyne'
        DOCKER_TAG = "latest"
    }

    stages {
        stage('Build Docker Images') {
            steps {
                script {
                    sh 'docker-compose -f docker-compose.yml build'
                }
            }
        }

        stage('Push to DockerHub') {
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") 
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                '''
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                sh "helm upgrade --install app-dev ./helm-chart --values=values-dev.yaml --namespace=dev"
            }
        }

        stage('Run Tests') {
            steps {
                sh "pytest tests/"
            }
        }

        stage('Deploy to QA') {
            steps {
                input message: 'Deploy to QA?'
                sh "helm upgrade --install app-qa ./helm-chart --values=values-qa.yaml --namespace=qa"
            }
        }

        stage('Deploy to Staging') {
            steps {
                input message: 'Deploy to Staging?'
                sh "helm upgrade --install app-staging ./helm-chart --values=values-staging.yaml --namespace=staging"
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to Production?'
                sh "helm upgrade --install app-prod ./helm-chart --values=values-prod.yaml --namespace=prod"
            }
        }
    }
}
