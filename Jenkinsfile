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
        }

        stage('Push to DockerHub') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") 
                DOCKER_ID = 'gdedeyne'
                DOCKER_IMAGE = 'jenkins_exam_cast_service' 
                DOCKER_TAG = 'latest' 
            }

            steps {
                script {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_ID --password-stdin
                    docker tag $DOCKER_IMAGE:$DOCKER_TAG $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Deploiement en dev'){
                environment
                {
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                }
                    steps {
                        script {
                        sh '''
                        rm -Rf .kube
                        mkdir .kube
                        ls
                        cat $KUBECONFIG > .kube/config
                        cp fastapi/values.yaml values.yml
                        cat values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install app app-dev --values=values.yml --namespace dev
                        '''
                        }
                    }

                }

        stage('Run Tests') {
            steps {
                echo('Test')
            }
        }

        stage('Deploiement en qa'){
                environment
                {
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                }
                    steps {
                        script {
                        sh '''
                        rm -Rf .kube
                        mkdir .kube
                        ls
                        cat $KUBECONFIG > .kube/config
                        cp fastapi/values.yaml values.yml
                        cat values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install app app-qa --values=values.yml --namespace qa
                        '''
                        }
                    }

                }

        stage('Deploiement en staging'){
                environment
                {
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                }
                    steps {
                        script {
                        sh '''
                        rm -Rf .kube
                        mkdir .kube
                        ls
                        cat $KUBECONFIG > .kube/config
                        cp fastapi/values.yaml values.yml
                        cat values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install app app-staging --values=values.yml --namespace staging
                        '''
                        }
                    }

                }

        stage('Deploiement en prod'){
                when {
                    branch 'main'
                }   
                environment
                {
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                }
                    steps {
                        script {
                        sh '''
                        rm -Rf .kube
                        mkdir .kube
                        ls
                        cat $KUBECONFIG > .kube/config
                        cp fastapi/values.yaml values.yml
                        cat values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install app app-staging --values=values.yml --namespace staging
                        '''
                        }
                    }

                }
    
}

