pipeline {
    agent {
        label 'laravel'
    }

    tools {
        git 'Default'
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
                sh 'ansible-playbook -i inventory.ini deploy.yml'
            }
        }
    }
}