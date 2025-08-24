pipeline {
    agent any

    environment {
        APP_DIR = "/home/ubuntu/myapp"
        EC2_USER = "ubuntu"
        EC2_HOST = "63.179.93.212"
        GIT_REPO = "git@github.com:arthisundarrajan/CICD.git"
        GIT_BRANCH = "master" // Make sure this matches your GitHub repo branch
    }

    stages {
        stage('Checkout') {
            steps {
                sshagent(credentials: ['ec2-ssh-key']) {
                    // Clone or pull repository in Jenkins workspace
                    sh """
                    if [ -d "$APP_DIR/.git" ]; then
                        cd $APP_DIR
                        git reset --hard
                        git pull origin $GIT_BRANCH
                    else
                        git clone -b $GIT_BRANCH $GIT_REPO $APP_DIR
                    fi
                    """
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                // Install Python dependencies locally in Jenkins workspace
                sh 'pip3 install -r requirements.txt || echo "No requirements.txt found"'
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ['ec2-ssh-key']) {
                    // Deploy to remote EC2 server
                    sh """
                    ssh -o StrictHostKeyChecking=no $EC2_USER@$EC2_HOST '
                        pkill -f app.py || true
                        mkdir -p $APP_DIR
                        cd $APP_DIR

                        if [ -d .git ]; then
                            git reset --hard
                            git pull origin $GIT_BRANCH
                        else
                            git clone -b $GIT_BRANCH $GIT_REPO .
                        fi

                        [ -f requirements.txt ] && pip3 install -r requirements.txt
                        nohup python3 app.py > app.log 2>&1 &
                    '
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed! Check console output for errors.'
        }
    }
}
