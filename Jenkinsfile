pipeline {
    agent any

    environment {
        DOCKERHUB_CREDS = credentials('DOCKER_HUB_PASS')
        IMAGE_MOVIE = "welchallch/movie-service"
        IMAGE_CAST  = "welchallch/cast-service"
        IMAGE_TAG   = "1.2"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Images') {
            steps {
                sh """
                docker build -t $IMAGE_MOVIE:$IMAGE_TAG movie-service/
                docker build -t $IMAGE_CAST:$IMAGE_TAG cast-service/
                """
            }
        }

        stage('Push Images') {
            steps {
                sh """
                echo $DOCKERHUB_CREDS_PSW | docker login -u $DOCKERHUB_CREDS_USR --password-stdin
                docker push $IMAGE_MOVIE:$IMAGE_TAG
                docker push $IMAGE_CAST:$IMAGE_TAG
                """
            }
        }

        stage('Deploy Dev') {
            steps {
                sh """
                helm upgrade --install app ./app/charts \
                --set movie.image.tag=$IMAGE_TAG \
                --set cast.image.tag=$IMAGE_TAG \
                -n dev
                """
            }
        }

        stage('Deploy Prod') {
            when { branch 'master' }
            steps {
                input message: "Deploy to Production?"
                sh """
                helm upgrade --install app ./app/charts \
                --set movie.image.tag=$IMAGE_TAG \
                --set cast.image.tag=$IMAGE_TAG \
                -n prod
                """
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}
