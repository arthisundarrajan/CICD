pipeline {
    agent any

    environment {
        APP_DIR = "/home/ubuntu/myapp"
        EC2_USER = "ubuntu"
        EC2_HOST = "63.179.93.212"
        GIT_REPO = "git@github.com:arthisundarrajan/CICD.git"
        GIT_BRANCH = "master" // Make sure this matches your repo
    }

    stages {
        stage('Checkout') {
            steps {
                // Use SSH key credentials for GitHub
                sshagent(credentials: ['ec2-ssh-key']) {
                    sh """
                    # Clone if not exists, else pull latest
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
                    sh """
                    ssh -o StrictHostKeyChecking=no $EC2_USER@$EC2_HOST '
                        # Stop any running app
                        pkill -f app.py || true

                        # Ensure app directory exists
                        mkdir -p $APP_DIR
                        cd $APP_DIR

                        # Clone repo if first time, else pull updates
                        if [ -d .git ]; then
                            git reset --hard
                            git pull origin $GIT_BRANCH
                        else
                            git clone -b $GIT_BRANCH $GIT_REPO .
                        fi

                        # Install dependencies on EC2
                        [ -f requirements.txt ] && pip3 install -r requirements.txt

                        # Start the app
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
