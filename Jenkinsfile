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
                git branch: "${env.BRANCH_NAME}",
                url: 'https://github.com/wassimCHall/Jenkins_devops_exams'
            }
        }

        stage('Build Images') {
            steps {
                sh 'docker build -t $IMAGE_MOVIE:${BUILD_NUMBER} movie-service/'
                sh 'docker build -t $IMAGE_CAST:${BUILD_NUMBER} cast-service/'
            }
        }

        stage('Push Images') {
            steps {
                sh 'docker login -u $DOCKERHUB_CREDS_USR -p $DOCKERHUB_CREDS_PSW'
                sh 'docker push $IMAGE_MOVIE:${BUILD_NUMBER}'
                sh 'docker push $IMAGE_CAST:${BUILD_NUMBER}'
            }
        }

        stage('Deploy Dev') {
            when { branch 'dev' }
            steps {
                sh 'helm upgrade --install app ./app/charts -n dev'
            }
        }

        stage('Deploy QA') {
            when { branch 'qa' }
            steps {
                sh 'helm upgrade --install app ./app/charts -n qa'
            }
        }

        stage('Deploy Staging') {
            when { branch 'staging' }
            steps {
                sh 'helm upgrade --install app ./app/charts -n staging'
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
