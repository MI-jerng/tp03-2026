pipeline {
    agent { label 'LaravelAgent' }

    stages {
        stage('Build & Test') {
            steps {
                echo 'Building and Testing...'
                checkout scm
                sh 'cp .env.example .env'
                sh 'composer install'
                sh 'npm install'
                sh 'php artisan key:generate'
                sh 'php artisan test'
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying to server..."
                // Use usernamePassword instead of string
                withCredentials([usernamePassword(credentialsId: 'server-ssh-login', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh "ansible-playbook -i inventory.ini deploy.yml -e ansible_password=${PASS}"
                }
            }
        }
    }

    post {
        always {
            script {
                def status = currentBuild.currentResult
                def message = "Build-${status}-Job-${env.JOB_NAME}-No-${env.BUILD_NUMBER}"
                
                // Matching your IDs: 'TELEGRAM_BOT_TOKEN' and 'TELEGRAM_CHAT_ID'
                try {
                    withCredentials([
                        string(credentialsId: 'TELEGRAM_BOT_TOKEN', variable: 'TOKEN'),
                        string(credentialsId: 'TELEGRAM_CHAT_ID', variable: 'CHAT_ID')
                    ]) {
                        sh "python3 -c \"import urllib.request; urllib.request.urlopen('https://api.telegram.org/bot${TOKEN}/sendMessage?chat_id=${CHAT_ID}&text=${message}')\""
                    }
                } catch (e) {
                    echo "Telegram notification failed: Ensure credentials match the code."
                }
            }
        }
    }
}