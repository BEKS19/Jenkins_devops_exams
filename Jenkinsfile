pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "glodyamba"
        CAST_IMAGE  = "${DOCKERHUB_USER}/cast-service:latest"
        MOVIE_IMAGE = "${DOCKERHUB_USER}/movie-service:latest"
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
                  docker build -t ${CAST_IMAGE}  ./cast-service
                  docker build -t ${MOVIE_IMAGE} ./movie-service
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
                      docker push ${CAST_IMAGE}
                      docker push ${MOVIE_IMAGE}
                    '''
                }
            }
        }

        /* =======================
           DEPLOY DEV
        ======================= */
        stage('Deploy DEV') {
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                    sh '''
                      helm upgrade --install cast-dev charts/cast \
                        --namespace dev --create-namespace
                      helm upgrade --install movie-dev charts/movie \
                        --namespace dev --create-namespace
                    '''
                }
            }
        }

        /* =======================
           DEPLOY QA
        ======================= */
        stage('Deploy QA') {
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                    sh '''
                      helm upgrade --install cast-qa charts/cast \
                        --namespace qa --create-namespace
                      helm upgrade --install movie-qa charts/movie \
                        --namespace qa --create-namespace
                    '''
                }
            }
        }

        /* =======================
           DEPLOY STAGING
        ======================= */
        stage('Deploy STAGING') {
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                    sh '''
                      helm upgrade --install cast-staging charts/cast \
                        --namespace staging --create-namespace
                      helm upgrade --install movie-staging charts/movie \
                        --namespace staging --create-namespace
                    '''
                }
            }
        }

        /* =======================
           DEPLOY PROD (MANUEL + MASTER)
        ======================= */
        stage('Deploy PROD') {
            when {
                branch 'master'
            }
            steps {
                input message: 'Déployer en PRODUCTION ?'
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                    sh '''
                      helm upgrade --install cast-prod charts/cast \
                        --namespace prod --create-namespace
                      helm upgrade --install movie-prod charts/movie \
                        --namespace prod --create-namespace
                    '''
                }
            }
        }
    }
}

