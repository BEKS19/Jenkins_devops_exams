pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "glodyamba"
        CAST_IMAGE_REPO  = "glodyamba/cast-service"
        MOVIE_IMAGE_REPO = "glodyamba/movie-service"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                  docker build -t ${CAST_IMAGE_REPO}:${IMAGE_TAG} ./cast-service
                  docker build -t ${MOVIE_IMAGE_REPO}:${IMAGE_TAG} ./movie-service
                '''
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                      echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                      docker push ${CAST_IMAGE_REPO}:${IMAGE_TAG}
                      docker push ${MOVIE_IMAGE_REPO}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy DEV') {
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                    sh '''
                      helm upgrade --install cast-dev charts/cast \
                        --namespace dev --create-namespace \
                        --set image.repository=${CAST_IMAGE_REPO} \
                        --set image.tag=${IMAGE_TAG}

                      helm upgrade --install movie-dev charts/movie \
                        --namespace dev --create-namespace \
                        --set image.repository=${MOVIE_IMAGE_REPO} \
                        --set image.tag=${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy QA') {
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                    sh '''
                      helm upgrade --install cast-qa charts/cast \
                        --namespace qa --create-namespace \
                        --set image.repository=${CAST_IMAGE_REPO} \
                        --set image.tag=${IMAGE_TAG}

                      helm upgrade --install movie-qa charts/movie \
                        --namespace qa --create-namespace \
                        --set image.repository=${MOVIE_IMAGE_REPO} \
                        --set image.tag=${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy STAGING') {
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                    sh '''
                      helm upgrade --install cast-staging charts/cast \
                        --namespace staging --create-namespace \
                        --set image.repository=${CAST_IMAGE_REPO} \
                        --set image.tag=${IMAGE_TAG}

                      helm upgrade --install movie-staging charts/movie \
                        --namespace staging --create-namespace \
                        --set image.repository=${MOVIE_IMAGE_REPO} \
                        --set image.tag=${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy PROD') {
            when {
                branch 'master'
            }
            steps {
                input message: 'Confirmer le déploiement en PRODUCTION'
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                    sh '''
                      helm upgrade --install cast-prod charts/cast \
                        --namespace prod --create-namespace \
                        --set image.repository=${CAST_IMAGE_REPO} \
                        --set image.tag=${IMAGE_TAG}

                      helm upgrade --install movie-prod charts/movie \
                        --namespace prod --create-namespace \
                        --set image.repository=${MOVIE_IMAGE_REPO} \
                        --set image.tag=${IMAGE_TAG}
                    '''
                }
            }
        }
    }
}

