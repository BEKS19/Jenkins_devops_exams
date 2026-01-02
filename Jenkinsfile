pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "glodyamba"
        CAST_IMAGE = "${DOCKERHUB_USER}/cast-service:latest"
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
                sh 'docker build -t $CAST_IMAGE ./cast-service'
                sh 'docker build -t $MOVIE_IMAGE ./movie-service'
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
                      docker push $CAST_IMAGE
                      docker push $MOVIE_IMAGE
                    '''
                }
            }
        }

        stage('Deploy DEV') {
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                    sh '''
                      export KUBECONFIG=$KUBECONFIG
                      helm upgrade --install cast-dev charts/cast --namespace dev --create-namespace
                      helm upgrade --install movie-dev charts/movie --namespace dev --create-namespace
                    '''
                }
            }
        }

        stage('Deploy QA') {
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                    sh '''
                      export KUBECONFIG=$KUBECONFIG
                      helm upgrade --install cast-qa charts/cast --namespace qa
                      helm upgrade --install movie-qa charts/movie --namespace qa
                    '''
                }
            }
        }

        stage('Deploy STAGING') {
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                    sh '''
                      export KUBECONFIG=$KUBECONFIG
                      helm upgrade --install cast-staging charts/cast --namespace staging
                      helm upgrade --install movie-staging charts/movie --namespace staging
                    '''
                }
            }
        }

        stage('Deploy PROD') {
            when {
                branch 'master'
            }
            steps {
                input message: 'Validation manuelle pour déploiement PROD'
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                    sh '''
                      export KUBECONFIG=$KUBECONFIG
                      helm upgrade --install cast-prod charts/cast --namespace prod
                      helm upgrade --install movie-prod charts/movie --namespace prod
                    '''
                }
            }
        }
    }
}

