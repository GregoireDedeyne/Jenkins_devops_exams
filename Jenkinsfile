pipeline {
    agent any

    environment {
        DOCKER_ID = 'gdedeyne'
        DOCKER_TAG = 'latest'
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
                    docker tag movie_service:latest $DOCKER_ID/movie_service:$DOCKER_TAG
                    docker push $DOCKER_ID/movie_service:$DOCKER_TAG

                    docker tag cast_service:latest $DOCKER_ID/cast_service:$DOCKER_TAG
                    docker push $DOCKER_ID/cast_service:$DOCKER_TAG
                    '''
                }
            }
        }       

        stage('Deploiement en dev') {
            environment {
                KUBECONFIG = credentials('config') // We retrieve kubeconfig from secret file called config saved on Jenkins
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
                    helm upgrade --install app-dev --values=values.yml --namespace dev
                    '''
                }
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Test'
            }
        }

        stage('Deploiement en qa') {
            environment {
                KUBECONFIG = credentials('config') // We retrieve kubeconfig from secret file called config saved on Jenkins
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
                    helm upgrade --install app-qa --values=values.yml --namespace qa
                    '''
                }
            }
        }

        stage('Deploiement en staging') {
            environment {
                KUBECONFIG = credentials('config') // We retrieve kubeconfig from secret file called config saved on Jenkins
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
                    helm upgrade --install app-staging --values=values.yml --namespace staging
                    '''
                }
            }
        }

        stage('Deploiement en prod') {
            when {
                branch 'main'
            }
            environment {
                KUBECONFIG = credentials('config') // We retrieve kubeconfig from secret file called config saved on Jenkins
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
                    helm upgrade --install app-staging --values=values.yml --namespace staging
                    '''
                }
            }
        }
    }
}
