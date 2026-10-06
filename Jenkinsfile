pipeline {

    environment {
        TARGET_IP = '34.236.150.119'
        SSH_USER  = 'ec2-user'
    }

    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/harshalfct/python-app.git'
            }
        }
        
        stage('Copy Application') {
            steps {
                sh '''
                    scp -o StrictHostKeyChecking=no \
                    -r . \
                    $SSH_USER@$TARGET_IP:/home/ec2-user/python-app
                '''
            }
        }

        stage('Install Python') {
            steps {
                sh '''
                    ssh -o StrictHostKeyChecking=no \
                    $SSH_USER@$TARGET_IP "
                        sudo yum update -y &&
                        sudo yum install -y python3
                    "
                '''
            }
        }

     

        stage('Install Dependencies') {
            steps {
                sh '''
                    ssh -o StrictHostKeyChecking=no \
                    $SSH_USER@$TARGET_IP "
                        cd /home/ec2-user/python-app &&
                        python3 -m venv .venv &&
                        .venv/bin/python -m pip install --upgrade pip &&
                        .venv/bin/python -m pip install -r requirements.txt
                    "
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    ssh -o StrictHostKeyChecking=no \
                    $SSH_USER@$TARGET_IP "
                        cd /home/ec2-user/python-app &&
                        .venv/bin/python -m unittest discover -s tests
                    "
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    ssh -o StrictHostKeyChecking=no \
                    $SSH_USER@$TARGET_IP "
                        cd /home/ec2-user/python-app &&
                        fuser -k 5000/tcp || true &&
                        JENKINS_NODE_COOKIE=dontKillMe \
                        nohup .venv/bin/python app.py > app.log 2>&1 &
                    "
                '''
            }
        }
    }
}
