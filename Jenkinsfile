pipeline {
    agent any

    environment {
        REMOTE_USER = 'mattis-efrei'
        REMOTE_HOST = 'ssh-mattis-efrei.alwaysdata.net'
        REMOTE_DIR  = '/home/mattis-efrei/www/'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Récupération du code source...'
                git branch: 'main',
                    url: 'https://github.com/mattisavec2t/landing-page-example'
                sh 'ls -la'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Déploiement sur Alwaysdata via SCP...'
                withCredentials([usernamePassword(
                    credentialsId: 'ALWAYSDATA_CREDENTIALS',
                    usernameVariable: 'SSH_USER',
                    passwordVariable: 'SSH_PASS'
                )]) {
                    sh """
                        sshpass -p "$SSH_PASS" scp \
                          -o StrictHostKeyChecking=no \
                          -r ./* \
                          $SSH_USER@${REMOTE_HOST}:${REMOTE_DIR}
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Déploiement réussi ✅'
        }
        failure {
            echo 'Échec du déploiement ❌'
        }
    }
}
