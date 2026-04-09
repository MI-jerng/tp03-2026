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
                echo 'Deploying to 178.128.93.188/sav-moeng...'
                // 'ssh-key-id' should be the ID of the SSH credential you created in Jenkins
                sshagent(['SSH_CREDENTIAL_ID_HERE']) {
                    sh 'ansible-playbook -i inventory.ini deploy.yml'
                }
            }
        }
    }
    post {
        always {
            script {
                def status = currentBuild.currentResult
                def icon = (status == 'SUCCESS') ? '✅' : '❌'
                
                sh """
                    sudo apt-get update && sudo apt-get install -y curl
                    curl -s -X POST https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage \
                    -d chat_id=${TELEGRAM_ID} \
                    -d text="${icon} Build ${status}: ${env.JOB_NAME} [${env.BUILD_NUMBER}]"
                """
            }
        }
    }
}