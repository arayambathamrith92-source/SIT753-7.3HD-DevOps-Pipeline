pipeline {
    agent any

    stages {

        stage('Build') {
            steps {
                echo 'Starting Node.js build...'

                bat 'node --version'
                bat 'npm --version'

                bat 'npm ci'
                bat 'npm run build'

                echo 'Build completed successfully.'
            }
        }

        stage('Test') {
            steps {
                echo 'Running automated tests...'

                bat 'npm test'

                echo 'Tests completed successfully.'
            }
        }
    }
}
