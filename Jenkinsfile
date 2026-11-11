
pipeline {
    agent any

    environment {
        APP_HOST = 'APPLICATION_EC2_PRIVATE_IP'
        SSH_CREDENTIALS = 'app-ec2-ssh'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Building static HTML application'

                sh '''
                    test -s index.html
                    echo "HTML application build successful"
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Running application tests'

                sh '''
                    grep -qi '<!doctype html' index.html
                    grep -q 'Hello, World!' index.html
                    grep -q 'Jenkins CI/CD Demo' index.html

                    echo "All application tests passed"
                '''
            }
        }

stage('Deploy') {
    steps {
        echo 'Deploying application to Amazon Linux'

        withCredentials([
            sshUserPrivateKey(
                credentialsId: 'app-ec2-ssh',
                keyFileVariable: 'SSH_KEY',
                usernameVariable: 'SSH_USER'
            )
        ]) {
            sh '''
                set -eu

                ssh -i "$SSH_KEY" \
                    -o StrictHostKeyChecking=yes \
                    $SSH_USER@$APP_HOST \
                    "sudo systemctl is-active nginx"

                scp -i "$SSH_KEY" \
                    -o StrictHostKeyChecking=yes \
                    index.html \
                    $SSH_USER@$APP_HOST:/tmp/index.html

                ssh -i "$SSH_KEY" \
                    -o StrictHostKeyChecking=yes \
                    $SSH_USER@$APP_HOST \
                    "sudo install -o root -g root -m 0644 \
                    /tmp/index.html \
                    /usr/share/nginx/html/index.html && \
                    rm -f /tmp/index.html"
            '''
        }
    }
}

stage('Smoke Test') {
    steps {
        echo 'Checking deployed application'

        sh '''
            set -eu

            curl --fail --silent --show-error \
                http://${APP_HOST}/ \
                -o response.html

            grep -q 'Hello, World!' response.html

            echo "SMOKE TEST PASSED"
        '''
    }
}


