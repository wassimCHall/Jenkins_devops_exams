pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        DOCKER_IMAGE_MOVIE = "welchallch/movie-service"
        DOCKER_IMAGE_CAST  = "welchallch/cast-service"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        KUBECONFIG = "/etc/rancher/k3s/k3s.yaml"
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
                    docker build -t $DOCKER_IMAGE_MOVIE:$IMAGE_TAG movie-service/
                    docker build -t $DOCKER_IMAGE_CAST:$IMAGE_TAG cast-service/
                '''
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'DOCKER_PASS',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKER_IMAGE_MOVIE:$IMAGE_TAG
                        docker push $DOCKER_IMAGE_CAST:$IMAGE_TAG
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy Dev') {
            when { branch 'dev' }
            steps {
                sh '''
                    helm upgrade --install app ./charts \
                    --create-namespace \
                    --set movie.image.tag=$IMAGE_TAG \
                    --set cast.image.tag=$IMAGE_TAG \
                    -n dev
                '''
            }
        }

        stage('Deploy QA') {
            when { branch 'qa' }
            steps {
                sh '''
                    helm upgrade --install app ./charts \
                    --create-namespace \
                    --set movie.image.tag=$IMAGE_TAG \
                    --set cast.image.tag=$IMAGE_TAG \
                    -n qa
                '''
            }
        }

        stage('Deploy Staging') {
            when { branch 'staging' }
            steps {
                sh '''
                    helm upgrade --install app ./charts \
                    --create-namespace \
                    --set movie.image.tag=$IMAGE_TAG \
                    --set cast.image.tag=$IMAGE_TAG \
                    -n staging
                '''
            }
        }

        stage('Deploy Production') {
            when { branch 'master' }
            steps {
                input message: "Confirmer le déploiement en PRODUCTION ?"
                sh '''
                    helm upgrade --install app ./charts \
                    --create-namespace \
                    --set movie.image.tag=$IMAGE_TAG \
                    --set cast.image.tag=$IMAGE_TAG \
                    -n prod
                '''
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo "Pipeline exécuté avec succès"
        }
        failure {
            echo "Pipeline échoué"
        }
    }
}
