pipeline {
    agent any

    environment {
        DOCKER_ID = 'gdedeyne'
        DOCKER_TAG = 'latest'
        NODE_PORT = "30007" 
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

        stage('Set NODE_PORT based on DEPLOY_ENV') {
            steps {
                script {
                
                    if (env.DEPLOY_ENV == 'dev') {
                        env.NODE_PORT = "30007"
                    } else if (env.DEPLOY_ENV == 'qa') {
                        env.NODE_PORT = "30008"
                    } else if (env.DEPLOY_ENV == 'staging') {
                        env.NODE_PORT = "30009"
                    } else if (env.DEPLOY_ENV == 'prod') {
                        env.NODE_PORT = "30010"
                    }
                    echo "Le port Node est maintenant : ${env.NODE_PORT}"
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
                    sed -i "s+nodePort:.*+nodePort: ${NODE_PORT}+g" values.yml
                    helm upgrade --install app-dev ./charts --values=values.yml --namespace dev
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
                    sed -i "s+nodePort:.*+nodePort: ${NODE_PORT}+g" values.yml
                    helm upgrade --install app-qa ./charts --values=values.yml --namespace qa
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
                    sed -i "s+nodePort:.*+nodePort: ${NODE_PORT}+g" values.yml
                    helm upgrade --install app-staging ./charts --values=values.yml --namespace staging
                    '''
                }
            }
        }

        stage('Deploiement en prod') {
            when {
                branch 'main'
            }
            environment {
                KUBECONFIG = credentials('config') 
                DEPLOY_ENV = 'prod' 
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
                    sed -i "s+nodePort:.*+nodePort: ${NODE_PORT}+g" values.yml
                    helm upgrade --install app-prod ./charts --values=values.yml --namespace prod
                    '''
                }
            }
        }
    }
}
