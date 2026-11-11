
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

                sshagent(credentials: [SSH_CREDENTIALS]) {

                    sh '''
                        set -eu

                        scp index.html \
                        ec2-user@${APP_HOST}:/tmp/index-${BUILD_NUMBER}.html

                        ssh ec2-user@${APP_HOST} \
                        "sudo install -o root -g root -m 0644 \
                        /tmp/index-${BUILD_NUMBER}.html \
                        /usr/share/nginx/html/index.html && \
                        sudo rm -f /tmp/index-${BUILD_NUMBER}.html"
                    '''
                }
            }
        }

        stage('Smoke Test') {
            steps {
                echo 'Checking live application'

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
    }

    post {
        success {
            echo 'APPLICATION DEPLOYED SUCCESSFULLY'
        }

        failure {
            echo 'PIPELINE FAILED - CHECK JENKINS CONSOLE OUTPUT'
        }
    }
}

