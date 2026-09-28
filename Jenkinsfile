pipeline {
    agent any
    environment {
        DOCKER_USERNAME = "charlesht"
        DOCKER_TAG = "${BUILD_ID}" // we will tag our images with the current build in order to increment the value by 1 with each new build
    }
    stages {
        stage('Build') {
            steps {
                sh '''
                docker build -t movie-service ./movie-service
                docker build -t cast-service ./cast-service
                '''
            }
        }

        stage('Test local via Docker compose') {
            steps {
                sh '''
                docker compose up -d --build
                sleep 15
                docker compose ps
                curl -is http://localhost:8080/api/v1/movies/
                curl -is http://localhost:8080/api/v1/casts/docs
                docker compose down
                '''
            }
        }

        stage('Tag') {
            steps {
                sh '''
                docker images | head -5
                docker tag movie-service $DOCKER_USERNAME/movie-service:$DOCKER_TAG
                docker tag cast-service $DOCKER_USERNAME/cast-service:$DOCKER_TAG
                docker images | head -5
                '''
            }
        }
    
        stage('Push') {
            environment{
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {
                sh '''
                docker login -u $DOCKER_USERNAME -p $DOCKER_PASS
                docker push $DOCKER_USERNAME/movie-service:$DOCKER_TAG
                docker push $DOCKER_USERNAME/cast-service:$DOCKER_TAG
                '''
            }
        }
        
        stage('Deploy movie-service dev') {
            environment { KUBECONFIG = credentials("config") }
            steps {
                sh '''
                    rm -Rf .kube && mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    helm upgrade --install movie-service charts --namespace dev \
                    --set image.tag=${DOCKER_TAG} \
                    --set service.nodePort=30001
                '''
            }
        }
    }
}
