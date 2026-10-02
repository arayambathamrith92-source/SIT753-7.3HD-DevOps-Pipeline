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

                echo 'Running automated application tests...'

                echo 'Checking JavaScript syntax...'
                bat 'node --check app.js'

                echo 'Checking generated build artifact...'
                bat 'if not exist public\\js\\bundle.js exit /b 1'

                echo 'Checking package configuration...'
                bat 'node -e "const fs=require(\"fs\"); const p=JSON.parse(fs.readFileSync(\"package.json\",\"utf8\")); if(!p.name || !p.version || !p.scripts || !p.scripts.build){process.exit(1)}; console.log(\"Package configuration test passed\")"'

                echo 'Checking required application files...'
                bat 'if not exist app.js exit /b 1'
                bat 'if not exist package.json exit /b 1'

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

                echo 'Running SonarCloud analysis...'

                withCredentials([
                    string(
                        credentialsId: 'sonarcloud-token',
                        variable: 'SONAR_TOKEN'
                    )
                ]) {

                    bat '''
                        npx --yes sonar-scanner ^
                        -Dsonar.projectKey=arayambathamrith92-source_SIT753-7.3HD-DevOps-Pipeline ^
                        -Dsonar.organization=arayambathamrith92-source ^
                        -Dsonar.host.url=https://sonarcloud.io ^
                        -Dsonar.token=%SONAR_TOKEN% ^
                        -Dsonar.sources=. ^
                        -Dsonar.exclusions=node_modules/**,public/js/bundle.js,exploit/**,sarif.json ^
                        -Dsonar.projectVersion=%BUILD_NUMBER%
                    '''
                }

                echo 'SONARCLOUD CODE QUALITY ANALYSIS COMPLETED.'
            }
        }
    }

    post {
        success {
            echo '========================================'
            echo 'PIPELINE SUCCESSFUL'
            echo '========================================'
            echo 'Build, Test and Code Quality stages passed.'
        }

        failure {
            echo '========================================'
            echo 'PIPELINE FAILED'
            echo '========================================'
            echo 'Please check the failed stage in the Jenkins console.'
            echo '========================================'
        }
    }
}

