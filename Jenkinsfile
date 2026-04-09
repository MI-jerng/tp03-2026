pipeline {
    agent { label 'LaravelAgent' }

    environment {
        // These are available during the stages
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
                def message = "Build-${status}-Job-${env.JOB_NAME}-No-${env.BUILD_NUMBER}"
                
                // Wrap in withCredentials so the variables exist in this block
                withCredentials([
                    string(credentialsId: 'TELEGRAM_TOKEN', variable: 'TOKEN'),
                    string(credentialsId: 'TELEGRAM_ID', variable: 'CHAT_ID')
                ]) {
                    sh """
                        python3 -c "import urllib.request; urllib.request.urlopen('https://api.telegram.org/bot${TOKEN}/sendMessage?chat_id=${CHAT_ID}&text=${message}')" || echo 'Notification failed'
                    """
                }
            }
        }
    }
}