pipeline {
    agent any

    environment {
        REMOTE_USER = 'ubuntu'
        REMOTE_HOST = 'ec2-13-38-219-17.eu-west-3.compute.amazonaws.com'
        DOCKER_IMAGE = 'mattisavec2t/landing-page:v1'
        CONTAINER_NAME = 'landing-page-container'
        DOCKER_HUB_CREDENTIALS_ID = 'docker-hub-credentials' 
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Récupération du code source...'
                git branch: 'master-aws',
                    url: 'https://github.com/mattisavec2t/landing-page-example'
                sh 'ls -la'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    // On utilise docker.withRegistry pour gérer le login/logout proprement
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_HUB_CREDENTIALS_ID}") {
                        echo "Construction et Push de l'image..."
                        def customImage = docker.build("${DOCKER_IMAGE}")
                        customImage.push()
                    }
                }
            }
        }

        stage('Deploy Docker Image') {
            steps {
                echo "Connexion à ${REMOTE_HOST} et déploiement de l'image ${DOCKER_IMAGE}..."
                
                sshagent(['landing-page-ssh']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} << 'EOF'
                            sudo docker pull ${DOCKER_IMAGE}

                            sudo docker stop ${CONTAINER_NAME} || true
                            sudo docker rm ${CONTAINER_NAME} || true

                            sudo docker run -d --name ${CONTAINER_NAME} -p 80:80 ${DOCKER_IMAGE}
                            
                            sudo docker image prune -f
EOF
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Déploiement Docker terminé avec succès ! ✅'
        }
        failure {
            echo 'Erreur lors du pipeline. ❌'
        }
    }
}