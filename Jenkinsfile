pipeline {
    agent any

    environment {
        APP_DIR = "/home/ubuntu/myapp"
        EC2_USER = "ubuntu"
        EC2_HOST = "63.179.93.212"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/arthisundarrajan/CICD.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'pip3 install -r requirements.txt'
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent (credentials: ['ec2-ssh-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no $EC2_USER@$EC2_HOST '
                        pkill -f app.py || true &&
                        cd $APP_DIR &&
                        git pull &&
                        pip3 install -r requirements.txt &&
                        nohup python3 app.py > app.log 2>&1 &
                    '
                    """
                }
            }
        }
    }
}
