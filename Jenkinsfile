pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'kalyansagar5/hotstar'
    }

    stages {

        stage('GIT CHECKOUT') {
            steps {
                git branch: 'main',
                    credentialsId: 'git_cred',
                    url: 'https://github.com/kalyansagar5/HOTSTAR-Project.git'
            }
        }

        stage('BUILD APP') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('BUILD DOCKER') {
            steps {
                sh 'docker build -t javaapp .'
            }
        }

        stage('DOCKER LOGIN & PUSH') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub_creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker tag javaapp $DOCKER_HUB_REPO:latest
                        docker push $DOCKER_HUB_REPO:latest
                    '''
                }
            }
        }

        stage('RUN CONTAINER') {
            steps {
                sh '''
                    docker stop javaappcontainer || true
                    docker rm javaappcontainer || true
                    docker run -d --name javaappcontainer -p 8085:8080 $DOCKER_HUB_REPO:latest
                    sleep 10
                    curl -f http://localhost:8085 || exit 1
                '''
            }
        }

        stage('DEPLOY TO SWARM') {
            steps {
                sh '''
                    docker swarm init || true
                    docker service rm java-service || true
                    docker service create \
                        --name java-service \
                        --publish 8086:8080 \
                        --replicas 2 \
                        $DOCKER_HUB_REPO:latest
                '''
            }
        }
    }
}
