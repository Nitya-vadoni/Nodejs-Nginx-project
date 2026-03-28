pipeline {
    agent any

    environment {
        DOCKER_USER = "nityavadoni"
        IMAGE_NODE = "${DOCKER_USER}/node-app"
        IMAGE_NGINX = "${DOCKER_USER}/nginx-proxy"
        VM_IP = "13.221.171.4"
    }

    stages {

        stage('Build Docker Images') {
            steps {
                sh 'docker build -t $IMAGE_NODE .'
                sh 'docker build -t $IMAGE_NGINX .'
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                }
            }
        }

        stage('Push Images') {
            steps {
                sh 'docker push $IMAGE_NODE'
                sh 'docker push $IMAGE_NGINX'
            }
        }

        stage('Deploy to VM') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@$VM_IP << EOF

                    docker stop nodeapp nginx || true
                    docker rm nodeapp nginx || true

                    docker pull $IMAGE_NODE
                    docker pull $IMAGE_NGINX

                    docker network create app-net || true

                    docker run -d --name nodeapp --network app-net $IMAGE_NODE

                    docker run -d --name nginx \
                        --network app-net \
                        -p 80:80 \
                        $IMAGE_NGINX

                    EOF
                    """
                }
            }
        }
    }
}
