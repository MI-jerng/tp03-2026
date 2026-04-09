pipeline {
    agent {
        label 'laravel'
    }

    environment {
        // This links the Jenkins Credential ID to a script variable
        TELEGRAM_TOKEN = credentials('TELEGRAM_BOT_TOKEN')
        TELEGRAM_ID    = credentials('TELEGRAM_CHAT_ID')
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building...'
                checkout scm

                echo 'Copy env file...'
                sh 'cp .env.example .env'

                echo 'Configuring database...'

                echo 'Initializing dependencies...'
                sh 'composer install'
                sh 'npm install'

                echo 'Generating application key...'
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
                echo 'Deploying to 178.128.93.188...'
                withCredentials([usernamePassword(credentialsId: 'server-ssh-login', 
                                 passwordVariable: 'SSH_PASS', 
                                 usernameVariable: 'SSH_USER')]) {
                    // Use 'sshpass' (usually pre-installed) or Ansible's built-in pass handling
                    sh "ansible-playbook -i inventory.ini deploy.yml -e 'ansible_password=${SSH_PASS}'"
                }
            }
        }
    }
    post {
        always {
            script {
                def status = currentBuild.currentResult
                def icon = (status == 'SUCCESS') ? '✅' : '❌'
                def msg = "${icon} Build ${status}: ${env.JOB_NAME} [${env.BUILD_NUMBER}]"
                
                // We use a Python one-liner because Python is almost always 
                // present on Ansible/Laravel agents even if wget is missing.
                sh "python3 -c \"import urllib.request; urllib.request.urlopen('https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage?chat_id=${TELEGRAM_ID}&text=${msg}')\" || echo 'Python notification failed'"
            }
        }
    }
}