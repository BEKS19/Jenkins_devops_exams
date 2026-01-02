
}pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "glodyamba"
        CAST_IMAGE  = "${DOCKERHUB_USER}/cast-service:latest"
        MOVIE_IMAGE = "${DOCKERHUB_USER}/movie-service:latest"
        KUBE_DIR = "/var/lib/jenkins/.kube"
        KUBE_CONFIG = "/var/lib/jenkins/.kube/config"
    }

    stages {

        /* =======================
           1. CHECKOUT DU CODE
           ======================= */
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        /* =======================
           2. BUILD DES IMAGES
           ======================= */
        stage('Build Docker Images') {
            steps {
                sh 'docker build -t $CAST_IMAGE ./cast-service'
                sh 'docker build -t $MOVIE_IMAGE ./movie-service'
            }
        }

        /* =======================
           3. PUSH DOCKERHUB
           ======================= */
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

        /* =======================
           4. DEPLOY DEV
           ======================= */
        stage('Deploy DEV') {
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                      mkdir -p $KUBE_DIR
                      cp $KUBECONFIG_FILE $KUBE_CONFIG
                      export KUBECONFIG=$KUBE_CONFIG

                      helm upgrade --install cast-dev charts/cast --namespace dev --create-namespace
                      helm upgrade --install movie-dev charts/movie --namespace dev --create-namespace
                    '''
                }
            }
        }

        /* =======================
           5. DEPLOY QA
           ======================= */
        stage('Deploy QA') {
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                      cp $KUBECONFIG_FILE $KUBE_CONFIG
                      export KUBECONFIG=$KUBE_CONFIG

                      helm upgrade --install cast-qa charts/cast --namespace qa --create-namespace
                      helm upgrade --install movie-qa charts/movie --namespace qa --create-namespace
                    '''
                }
            }
        }

        /* =======================
           6. DEPLOY STAGING
           ======================= */
        stage('Deploy STAGING') {
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                      cp $KUBECONFIG_FILE $KUBE_CONFIG
                      export KUBECONFIG=$KUBE_CONFIG

                      helm upgrade --install cast-staging charts/cast --namespace staging --create-namespace
                      helm upgrade --install movie-staging charts/movie --namespace staging --create-namespace
                    '''
                }
            }
        }

        /* =======================
           7. DEPLOY PROD (MANUEL)
           ======================= */
        stage('Deploy PROD') {
            when {
                branch 'master'
            }
            steps {
                input message: 'Validation manuelle requise pour le déploiement en PRODUCTION'

                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                      cp $KUBECONFIG_FILE $KUBE_CONFIG
                      export KUBECONFIG=$KUBE_CONFIG

                      helm upgrade --install cast-prod charts/cast --namespace prod --create-namespace
                      helm upgrade --install movie-prod charts/movie --namespace prod --create-namespace
                    '''
                }
            }
        }
    }
}

