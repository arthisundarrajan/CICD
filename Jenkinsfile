pipeline {
    agent any

    environment {
        APP_DIR = "/home/ubuntu/myapp"
        EC2_USER = "ubuntu"
        EC2_HOST = "63.179.93.212"
        GIT_REPO = "git@github.com:arthisundarrajan/CICD.git" // SSH URL
        GIT_BRANCH = "master" // Correct branch name
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                # Install Python dependencies locally in Jenkins workspace
                if [ -f requirements.txt ]; then
                    pip3 install -r requirements.txt
                else
                    echo "No requirements.txt found"
                fi
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ['ec2-ssh-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no $EC2_USER@$EC2_HOST '
                        # Kill any existing app process
                        pkill -f app.py || true

                        # Ensure app directory exists
                        mkdir -p $APP_DIR
                        cd $APP_DIR

                        # Pull latest code from GitHub
                        if [ -d .git ]; then
                            git reset --hard
                            git pull origin ${GIT_BRANCH}
                        else
                            git clone -b ${GIT_BRANCH} ${GIT_REPO} .
                        fi

                        # Install dependencies on EC2
                        if [ -f requirements.txt ]; then
                            pip3 install -r requirements.txt
                        fi

                        # Start the application
                        nohup python3 app.py > app.log 2>&1 &
                    '
                    """
                }
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed! Check console output for errors.'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
    }
}
