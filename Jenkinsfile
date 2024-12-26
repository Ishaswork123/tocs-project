pipeline {
    agent any

    environment {
        NODE_SERVER_IP = '13.49.77.34'  // Node.js server's IP
        SSH_CREDENTIAL_ID = 'apache'     // Jenkins SSH Credential ID
        DEPLOY_PATH = '/home/ubuntu/nodejs-app/'  // Deployment path on the server
        LOCAL_FILES = 'index.js views/home.ejs package.json' // Files to deploy
    }

    stages {
        stage('Setup SSH Known Hosts') {
            steps {
                sshagent(["${SSH_CREDENTIAL_ID}"]) {
                    script {
                        echo "Adding ${NODE_SERVER_IP} to known_hosts..."
                        sh "ssh-keyscan -H ${NODE_SERVER_IP} >> ~/.ssh/known_hosts"
                    }
                }
            }
        }

        stage('Deploy Node.js Files') {
            steps {
                sshagent(["${SSH_CREDENTIAL_ID}"]) {
                    script {
                        echo "Deploying files to ${NODE_SERVER_IP}..."

                        // Ensure the deployment directory exists on the server
                        sh "ssh ubuntu@${NODE_SERVER_IP} 'mkdir -p ${DEPLOY_PATH}'"

                        // Copy the files to the server
                        sh "scp -r ${LOCAL_FILES} ubuntu@${NODE_SERVER_IP}:${DEPLOY_PATH}"
                    }
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sshagent(["${SSH_CREDENTIAL_ID}"]) {
                    script {
                        echo "Installing dependencies on ${NODE_SERVER_IP}..."

                        // Install Node.js dependencies
                        sh "ssh ubuntu@${NODE_SERVER_IP} 'cd ${DEPLOY_PATH} && npm install'"
                    }
                }
            }
        }

        stage('Start Node.js Application') {
            steps {
                sshagent(["${SSH_CREDENTIAL_ID}"]) {
                    script {
                        echo "Starting the Node.js application on ${NODE_SERVER_IP}..."

                        // Start the Node.js application using pm2 or node
                        sh "ssh ubuntu@${NODE_SERVER_IP} 'cd ${DEPLOY_PATH} && pm2 start index.js --name nodejs-app || node index.js &'"
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    echo "Verifying deployment on ${NODE_SERVER_IP}..."

                    // Verify the application is running and accessible
                    sh "curl http://${NODE_SERVER_IP}:3000"  // Assuming the app runs on port 3000
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully!'
        }
        failure {
            echo 'Deployment failed. Check the logs for more details.'
        }
    }
}
