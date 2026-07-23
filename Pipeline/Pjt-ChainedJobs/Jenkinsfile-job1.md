pipeline {
    agent any

    stages {
        stage('Deploy') {
            steps {
                sh 'ssh -o StrictHostKeyChecking=no sarah@stapp01 "cd /var/www/html && git pull origin master"'
            }
        }
    }
}
