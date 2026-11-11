
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        // Replace with your APPLICATION EC2 PRIVATE IP
        APP_HOST = '10.0.1.100'

        // Jenkins SSH credential ID
        SSH_CREDENTIALS = 'app-ec2-ssh'

        // Nginx deployment directory
        DEPLOY_DIR = '/usr/share/nginx/html'
    }

    stages {

        // STAGE 1: CHECKOUT
        stage('Checkout') {
            steps {
                echo 'Checking out application from GitHub'

                checkout scm

                sh '''
                    echo "Current branch:"
                    git branch --show-current

                    echo "Application files:"
                    ls -la
                '''
            }
        }

        // STAGE 2: BUILD
        stage('Build') {
            steps {
                echo 'Building HTML application'

                sh '''
                    set -eu

                    if [ ! -s index.html ]; then
                        echo "ERROR: index.html not found or empty"
                        exit 1
                    fi

                    echo "HTML application build successful"
                '''
            }
        }

        // STAGE 3: TEST
        stage('Test') {
            steps {
                echo 'Running HTML application tests'

                sh '''
                    set -eu

                    grep -qi '<!DOCTYPE html' index.html
                    grep -q 'Hello, World!' index.html
                    grep -q 'Jenkins CI/CD Demo' index.html

                    echo "All application tests passed"
                '''
            }
        }

        // STAGE 4: DEPLOY
        stage('Deploy') {
            steps {
                echo 'Deploying application to Amazon Linux EC2'

                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'app-ec2-ssh',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {

                    sh '''
                        set -eu

                        SSH_OPTIONS="-i $SSH_KEY -o BatchMode=yes -o StrictHostKeyChecking=yes"

                        echo "Checking SSH connectivity"

                        ssh $SSH_OPTIONS \
                            "$SSH_USER@$APP_HOST" \
                            "sudo systemctl is-active --quiet nginx"

                        echo "Copying application to EC2"

                        scp $SSH_OPTIONS \
                            index.html \
                            "$SSH_USER@$APP_HOST:/tmp/index-${BUILD_NUMBER}.html"

                        echo "Installing application in Nginx directory"

                        ssh $SSH_OPTIONS \
                            "$SSH_USER@$APP_HOST" \
                            "sudo install -o root -g root -m 0644 \
                            /tmp/index-${BUILD_NUMBER}.html \
                            $DEPLOY_DIR/index.html && \
                            rm -f /tmp/index-${BUILD_NUMBER}.html"

                        echo "Application deployment successful"
                    '''
                }
            }
        }

        // STAGE 5: SMOKE TEST
        stage('Smoke Test') {
            steps {
                echo 'Testing deployed application'

                sh '''
                    set -eu

                    echo "Checking HTTP response"

                    curl --fail \
                        --silent \
                        --show-error \
                        --max-time 15 \
                        "http://${APP_HOST}/" \
                        -o response.html

                    echo "Validating deployed HTML"

                    grep -q 'Hello, World!' response.html

                    grep -q 'Jenkins CI/CD Demo' response.html

                    echo "SMOKE TEST PASSED"
                '''
            }
        }
    }

    post {
        success {
            echo '''
            =====================================
              APPLICATION DEPLOYED SUCCESSFULLY
            =====================================
            '''
        }

        failure {
            echo '''
            =====================================
              PIPELINE FAILED
              CHECK JENKINS CONSOLE OUTPUT
            =====================================
            '''
        }

        always {
            echo 'Jenkins pipeline execution completed'
        }
    }
}

