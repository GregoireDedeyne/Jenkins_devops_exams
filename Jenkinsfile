pipeline {
    agent any

    environment {
        DOCKER_ID = 'gdedeyne'
        DOCKER_TAG = 'latest'
    }
    stages {
        stage('Check Branch') {
            steps {
                script {
                    checkout scm  // Récupère les informations du dépôt
                    echo "Current branch is: ${env.GIT_BRANCH}"
                }
            }
        }
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
            environment {
                DOCKER_PASS = credentials('DOCKER_HUB_PASS')
                DOCKER_ID = 'gdedeyne'
                DOCKER_TAG = 'latest'
            }
            steps {
                script {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_ID --password-stdin
                    docker tag jenkins_exam_movie_service:latest $DOCKER_ID/movie_service:$DOCKER_TAG
                    docker push $DOCKER_ID/movie_service:$DOCKER_TAG

                    docker tag jenkins_exam_cast_service:latest $DOCKER_ID/cast_service:$DOCKER_TAG
                    docker push $DOCKER_ID/cast_service:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Deploiement en dev') {
            environment {
                KUBECONFIG = credentials('config')
                DEPLOY_ENV = 'dev'
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp charts/values.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    sed -i "s+nodePort:.*+nodePort: 30007+g" values.yml
                    helm upgrade --install app-${DEPLOY_ENV} ./charts --values=values.yml --namespace ${DEPLOY_ENV}
                    '''
                }
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Test'
            }
        }

        stage('Deploiement en QA') {
            environment {
                KUBECONFIG = credentials('config')
                DEPLOY_ENV = 'qa'
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp charts/values.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    sed -i "s+nodePort:.*+nodePort: 30008+g" values.yml
                    helm upgrade --install app-${DEPLOY_ENV} ./charts --values=values.yml --namespace ${DEPLOY_ENV}
                    '''
                }
            }
        }

        stage('Deploiement en staging') {
            environment {
                KUBECONFIG = credentials('config')
                DEPLOY_ENV = 'staging'
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp charts/values.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    sed -i "s+nodePort:.*+nodePort: 30009+g" values.yml
                    helm upgrade --install app-${DEPLOY_ENV} ./charts --values=values.yml --namespace ${DEPLOY_ENV}
                    '''
                }
            }
        }
        stage('Check Branch') {
            steps {
                script {
                    echo "Current branch is: ${env.BRANCH_NAME}"
                }
            }
        }
        stage('Deploiement en prod') {
            when {
                branch pattern: '^(master|main)$', comparator: 'REGEXP'
            }
            environment {
                KUBECONFIG = credentials('config')
                DEPLOY_ENV = 'prod'
            }
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in production?', ok: 'Yes'
                }
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp charts/values.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    sed -i "s+nodePort:.*+nodePort: 30010+g" values.yml  // Remplacement pour prod
                    helm upgrade --install app-${DEPLOY_ENV} ./charts --values=values.yml --namespace ${DEPLOY_ENV}
                    '''
                }
            }
        }
    }
}
