pipeline {
    agent any
    environment {
        SERVER_IP = '192.168.1.186'
        APP_PATH = '/var/www/email-react-app'
        DEPLOY_PATH = '/var/www/email-react-app-build'
        BRANCH = 'main'
    }
    stages {
        stage('Send Approval Email') {
            steps {
                mail to: 'jadhavrohitbh@gmail.com',
                     subject: "Approval Required for Deployment",
                     body: """
Pipeline triggered.
Approve here:
${env.BUILD_URL}input
"""
            }
        }
        stage('Wait for Approval') {
            steps {
                input message: "Approve deployment?", ok: "Deploy"
            }
        }
        stage('Deploy to Server') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'ubuntu-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh '''
                    echo "Connecting to server..."
                    ssh -i $SSH_KEY -o StrictHostKeyChecking=no $SSH_USER@$SERVER_IP << 'EOF'
                    export PATH=/usr/bin:/usr/local/bin:$PATH
                    cd /var/www/email-react-app
                    set -e
                    export NVM_DIR="$HOME/.nvm"
                    [ -s "$NVM_DIR/nvm.sh" ] && \\. "$NVM_DIR/nvm.sh"
                    nvm use 18
                    echo "Pulling latest code..."
                    git reset --hard
                    git pull origin main
                    echo "Installing dependencies..."
                    npm install
                    echo "Building React app..."
                    npm run build
                    echo "Deploying to Nginx..."
                    sudo mkdir -p /var/www/email-react-app-build
                    sudo rm -rf /var/www/email-react-app-build/*
                    sudo cp -r build/* /var/www/email-react-app-build/
                    echo "Restarting Nginx..."
                    sudo systemctl restart nginx
                    echo "Deployment completed successfully"
EOF
                    '''
                }
            }
        }
    }
}
