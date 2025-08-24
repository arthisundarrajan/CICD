pipeline {
    agent any

    environment {
        APP_DIR = "/home/ubuntu/myapp"
        EC2_USER = "ubuntu"
        EC2_HOST = "63.179.93.212"
        GIT_REPO = "git@github.com:arthisundarrajan/CICD.git" // SSH URL
        GIT_BRANCH = "master" // Make sure this matches your repo branch
    }

    stages {
        stage('Checkout') {
            steps {
                sshagent(credentials: ['ec2-ssh-key']) {
                    // Use SSH URL to clone repo using Jenkins SSH key
                    sh "git clone -b ${GIT_BRANCH} ${GIT_REPO} || (cd CICD && git reset --hard && git pull origin ${GIT_BRANCH})"
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                # Check for requirements.txt before installing
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
                    ssh -o StrictHostKeyChecking=no
