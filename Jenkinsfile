pipeline {

    environment {
        TARGET_IP = '34.236.150.119'
        CRED_ID   = 'ec2-target-key'
    }

    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Rajpardeshi205/python-app.git'
            }
        }

        stage('Install Python') {
            steps {
                sh '''
                    sudo yum update -y
                    sudo yum install -y python3
                '''
            }
        }

        stage('Copy Application') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: "${CRED_ID}",
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh '''
                        scp -i $SSH_KEY \
                        -o StrictHostKeyChecking=no \
                        -r . \
                        $SSH_USER@$TARGET_IP:/home/ec2-user/python-app
                    '''
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: "${CRED_ID}",
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh '''
                        ssh -i $SSH_KEY \
                        -o StrictHostKeyChecking=no \
                        $SSH_USER@$TARGET_IP "
                            cd /home/ec2-user/python-app &&
                            python3 -m venv .venv &&
                            .venv/bin/python -m pip install --upgrade pip &&
                            .venv/bin/python -m pip install -r requirements.txt
                        "
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: "${CRED_ID}",
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh '''
                        ssh -i $SSH_KEY \
                        -o StrictHostKeyChecking=no \
                        $SSH_USER@$TARGET_IP "
                            cd /home/ec2-user/python-app &&
                            .venv/bin/python -m unittest discover -s tests
                        "
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: "${CRED_ID}",
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh '''
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no \
                        $SSH_USER@$TARGET_IP '
                            cd /home/$USER/python-app
                            fuser -k 5000/tcp >/dev/null 2>&1 || true
                            setsid nohup .venv/bin/python app.py > app.log 2>&1 < /dev/null &
                            sleep 3
                            curl -fsS http://localhost:5000/ >/dev/null && echo "App is up"
                        '
                    '''
                }
            }
        }
    }
}
