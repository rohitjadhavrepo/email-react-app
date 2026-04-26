pipeline {
    agent any
    environment {
        SERVER_IP = '192.168.1.186'
        APP_PATH = '/var/www/email-react-app'   // your project path on server
        DEPLOY_PATH = '/var/www/email-react-app-build'           // nginx root
        BRANCH = 'main'
    }
    stages {
        stage('Send Approval Email') {
            steps {
                mail to: '[jadhavrohitbh@gmail.com](mailto:jadhavrohitbh@gmail.com)',
                     subject: "Approval Required for Deployment",
                     body: """
Pipeline triggered.
Approve here:
${[env.BUILD](http://env.BUILD)_URL}input
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
                    ssh -i $SSH_KEY -o StrictHostKeyChecking=no $SSH_USER@$SERVER_IP << EOF

                    export PATH=/usr/bin:/usr/local/bin:$PATH
                    cd $APP_PATH

                    set -e
                    # Load NVM manually
                    export NVM_DIR="$HOME/.nvm"
                    [ -s "$NVM_DIR/[nvm.sh](http://nvm.sh)" ] && \\. "$NVM_DIR/[nvm.sh](http://nvm.sh)"
                    # Use Node 18
                    nvm use 18                    
                    echo "Pulling latest code..."
                    git reset --hard
                    git pull origin $BRANCH
                    echo "Installing dependencies..."
                    npm install
                    echo "Building React app..."
                    npm run build
                    echo "Deploying to Nginx..."
                    sudo mkdir -p $DEPLOY_PATH
                    sudo rm -rf $DEPLOY_PATH/*
                    sudo cp -r build/* $DEPLOY_PATH/
                    echo "Restarting Nginx..."
                    sudo systemctl restart nginx
                    echo "Deployment completed successfully "
                    EOF
                    '''
                }
            }
        }
    }
}
