pipeline {
    agent any

    stages {

        stage('Build') {
            steps {
                echo '========================================'
                echo 'BUILD STAGE'
                echo '========================================'

                bat 'node --version'
                bat 'npm --version'

                echo 'Installing project dependencies...'
                bat 'npm ci'

                echo 'Building the Node.js application...'
                bat 'npm run build'

                echo 'Build completed successfully.'
            }
        }

        stage('Test') {
            steps {
                echo '========================================'
                echo 'TEST STAGE'
                echo '========================================'

                echo 'Running automated application tests...'

                bat 'npx mocha "tests/**/*.spec.js"'

                echo 'Automated tests completed successfully.'
            }
        }
    }
}
