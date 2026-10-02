pipeline {
    agent any

    stages {

        // ============================================================
        // STAGE 1 - BUILD
        // ============================================================
        stage('Build') {
            steps {
                echo '========================================'
                echo 'STAGE 1: BUILD'
                echo '========================================'

                echo 'Checking Node.js and npm versions...'
                bat 'node --version'
                bat 'npm --version'

                echo 'Installing project dependencies...'
                bat 'npm ci'

                echo 'Building the Node.js application...'
                bat 'npm run build'

                echo 'BUILD COMPLETED SUCCESSFULLY.'
            }
        }


        // ============================================================
        // STAGE 2 - TEST
        // ============================================================
        stage('Test') {
            steps {
                echo '========================================'
                echo 'STAGE 2: TEST'
                echo '========================================'

                echo 'Running automated tests...'

                bat 'npm test'

                echo 'ALL AUTOMATED TESTS PASSED.'
            }
        }


        // ============================================================
        // STAGE 3 - CODE QUALITY
        // ============================================================
        stage('Code Quality') {
            steps {
                echo '========================================'
                echo 'STAGE 3: CODE QUALITY'
                echo '========================================'

                echo 'Running SonarCloud code quality analysis...'

                withCredentials([
                    string(
                        credentialsId: 'sonarcloud-token',
                        variable: 'SONAR_TOKEN'
                    )
                ]) {

                    bat '''
                        npx sonar-scanner ^
                        -Dsonar.projectKey=arayambathamrith92-source_SIT753-7.3HD-DevOps-Pipeline ^
                        -Dsonar.organization=arayambathamrith92-source ^
                        -Dsonar.host.url=https://sonarcloud.io ^
                        -Dsonar.token=%SONAR_TOKEN% ^
                        -Dsonar.sources=. ^
                        -Dsonar.tests=tests ^
                        -Dsonar.test.inclusions=tests/**/*.js ^
                        -Dsonar.exclusions=node_modules/**,public/js/bundle.js ^
                        -Dsonar.sourceEncoding=UTF-8
                    '''
                }

                echo 'SONARCLOUD CODE QUALITY ANALYSIS COMPLETED.'
            }
        }
    }

    post {
        success {
            echo '========================================'
            echo 'PIPELINE COMPLETED SUCCESSFULLY'
            echo '========================================'
        }

        failure {
            echo '========================================'
            echo 'PIPELINE FAILED'
            echo 'Please check the failed stage in the Jenkins console.'
            echo '========================================'
        }
    }
}
