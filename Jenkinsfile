pipeline {
    agent any

    environment {
        DOCKERHUB_CREDS = credentials('dockerhub')
        IMAGE_MOVIE = "welchallch/movie-service"
        IMAGE_CAST = "welchallch/cast-service"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Images') {
            steps {
                sh 'docker build -t $IMAGE_MOVIE:${BUILD_NUMBER} app/movie-service/'
                sh 'docker build -t $IMAGE_CAST:${BUILD_NUMBER} app/cast-service/'
            }
        }

        stage('Push Images') {
            steps {
                sh 'echo $DOCKERHUB_CREDS_PSW | docker login -u $DOCKERHUB_CREDS_USR --password-stdin'
                sh 'docker push $IMAGE_MOVIE:${BUILD_NUMBER}'
                sh 'docker push $IMAGE_CAST:${BUILD_NUMBER}'
            }
        }

        stage('Deploy Dev') {
            steps {
                sh 'helm upgrade --install app ./app/charts -n dev'
            }
        }

        stage('Deploy Prod') {
            when { branch 'master' }
            steps {
                input message: "Deploy to Production?"
                sh 'helm upgrade --install app ./app/charts -n prod'
            }
        }
    }
}
