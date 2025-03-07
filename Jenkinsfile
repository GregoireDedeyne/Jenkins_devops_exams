pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'gdedeyne/datascientestapi_exam'
        DOCKER_TAG = "latest"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                sh "docker build -t $DOCKER_IMAGE:$DOCKER_TAG ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                withDockerRegistry([credentialsId: 'DOCKER_HUB_PASS', url: '']) {
                    sh "docker push $DOCKER_IMAGE:$DOCKER_TAG"
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

    post {
        success {
            echo "Déploiement réussi"
        }
        failure {
            echo "Echec"
        }
    }
}