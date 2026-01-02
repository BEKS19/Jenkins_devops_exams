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
                sh 'docker push $CAST_IMAGE'
                sh 'docker push $MOVIE_IMAGE'
            }
        }

        stage('Deploy DEV') {
            steps {
                sh 'helm upgrade --install cast-dev charts/cast --namespace dev'
                sh 'helm upgrade --install movie-dev charts/movie --namespace dev'
            }
        }

        stage('Deploy QA') {
            steps {
                sh 'helm upgrade --install cast-qa charts/cast --namespace qa'
                sh 'helm upgrade --install movie-qa charts/movie --namespace qa'
            }
        }

        stage('Deploy STAGING') {
            steps {
                sh 'helm upgrade --install cast-staging charts/cast --namespace staging'
                sh 'helm upgrade --install movie-staging charts/movie --namespace staging'
            }
        }

        stage('Deploy PROD') {
            when {
                branch 'master'
            }
            input {
                message "Déployer en production ?"
                ok "Déployer"
            }
            steps {
                sh 'helm upgrade --install cast-prod charts/cast --namespace prod'
                sh 'helm upgrade --install movie-prod charts/movie --namespace prod'
            }
        }
    }
}
