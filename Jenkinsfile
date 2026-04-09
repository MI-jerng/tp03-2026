pipeline {
    agent { label 'LaravelAgent' }

    environment {
        TELEGRAM_TOKEN = credentials('TELEGRAM_TOKEN')
        TELEGRAM_ID = credentials('TELEGRAM_ID')
        SSH_PASS = credentials('SSH_PASS')
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building...'
                checkout scm
                sh 'cp .env.example .env'
                sh 'composer install'
                sh 'npm install'
                sh 'php artisan key:generate'
            }
        }

        stage('Test') {
            steps {
                echo 'Testing...'
                sh 'php artisan test'
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying to server..."
                withCredentials([string(credentialsId: 'SSH_PASS', variable: 'SSH_PASS')]) {
                    sh "ansible-playbook -i inventory.ini deploy.yml -e ansible_password=${SSH_PASS}"
                }
            }
        }
    }

    post {
        always {
            script {
                def status = currentBuild.currentResult
                // Keeping the message simple to avoid URL encoding errors in Python
                def message = "Build-${status}-Job-${env.JOB_NAME}-Number-${env.BUILD_NUMBER}"
                
                sh """
                    python3 -c "import urllib.request; urllib.request.urlopen('https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage?chat_id=${TELEGRAM_ID}&text=${message}')" || echo 'Notification failed'
                """
            }
        }
    }
}