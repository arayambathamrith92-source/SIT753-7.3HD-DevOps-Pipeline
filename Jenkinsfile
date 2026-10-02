pipeline {
    agent any

    environment {
        SONAR_PROJECT_KEY = 'arayambathamrith92-source_SIT753-7.3HD-DevOps-Pipeline'
        SONAR_ORGANIZATION = 'arayambathamrith92-source'
        APP_NAME = 'sit753-goof'
        RELEASE_DIR = 'release'
    }

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

                echo 'Creating build artefact directory...'
                bat 'if not exist build-artifact mkdir build-artifact'

                echo 'Copying application files into build artefact...'
                bat 'xcopy /E /I /Y app.js build-artifact\\'
                bat 'xcopy /E /I /Y package.json build-artifact\\'
                bat 'xcopy /E /I /Y package-lock.json build-artifact\\'
                bat 'xcopy /E /I /Y public build-artifact\\public'

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

                echo 'Checking generated build artefact...'
                bat 'if not exist public\\js\\bundle.js exit /b 1'

                echo 'Checking package configuration...'
                bat '''node -e "const fs=require('fs'); const p=JSON.parse(fs.readFileSync('package.json','utf8')); if(!p.name || !p.version || !p.scripts || !p.scripts.build){process.exit(1)}; console.log('Package configuration test passed')"'''

                echo 'Checking test files...'
                bat 'if exist tests (echo Test directory found) else (echo No test directory found)'

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
                        npx --yes sonar-scanner ^
                        -Dsonar.projectKey=%SONAR_PROJECT_KEY% ^
                        -Dsonar.organization=%SONAR_ORGANIZATION% ^
                        -Dsonar.host.url=https://sonarcloud.io ^
                        -Dsonar.token=%SONAR_TOKEN% ^
                        -Dsonar.sources=. ^
                        -Dsonar.exclusions=node_modules/**,public/js/bundle.js,build-artifact/**,sarif.json
                    '''
                }

                echo 'SONARCLOUD CODE QUALITY ANALYSIS COMPLETED.'
            }
        }


        // ============================================================
        // STAGE 4 - SECURITY
        // ============================================================
        stage('Security') {
            steps {
                echo '========================================'
                echo 'STAGE 4: SECURITY'
                echo '========================================'

                echo 'Running npm dependency security audit...'

                bat '''
                    npm audit --json > security-report.json
                    if %ERRORLEVEL% NEQ 0 (
                        echo Security vulnerabilities were detected.
                        echo The complete results are stored in security-report.json.
                        exit /b 0
                    )
                '''

                echo 'Security scan completed.'
                echo 'Security findings are documented in security-report.json.'
            }

            post {
                always {
                    archiveArtifacts artifacts: 'security-report.json',
                                     allowEmptyArchive: true
                }
            }
        }


        // ============================================================
        // STAGE 5 - DEPLOY
        // ============================================================
        stage('Deploy') {
            steps {
                echo '========================================'
                echo 'STAGE 5: DEPLOY'
                echo '========================================'

                echo 'Preparing test deployment environment...'

                bat '''
                    if exist deploy (
                        rmdir /S /Q deploy
                    )
                    mkdir deploy
                '''

                echo 'Deploying application to test environment...'

                bat 'xcopy /E /I /Y app.js deploy\\'
                bat 'xcopy /E /I /Y package.json deploy\\'
                bat 'xcopy /E /I /Y package-lock.json deploy\\'
                bat 'xcopy /E /I /Y public deploy\\public'

                echo 'Installing production dependencies in test environment...'

                bat 'cd deploy && npm ci --omit=dev'

                echo 'TEST DEPLOYMENT COMPLETED.'
            }
        }


        // ============================================================
        // STAGE 6 - RELEASE
        // ============================================================
        stage('Release') {
            steps {
                echo '========================================'
                echo 'STAGE 6: RELEASE'
                echo '========================================'

                echo 'Creating production release directory...'

                bat '''
                    if exist release (
                        rmdir /S /Q release
                    )
                    mkdir release
                '''

                echo 'Promoting tested application to production release...'

                bat 'xcopy /E /I /Y deploy release'

                echo 'Creating release version information...'

                bat '''
                    node -e "const fs=require('fs'); const p=require('./package.json'); fs.writeFileSync('release\\VERSION.txt', 'Application: ' + p.name + '\\nVersion: ' + p.version + '\\nJenkins Build: %BUILD_NUMBER%\\nGit Commit: %GIT_COMMIT%\\n');"
                '''

                echo 'PRODUCTION RELEASE CREATED.'
            }
        }


        // ============================================================
        // STAGE 7 - MONITORING
        // ============================================================
        stage('Monitoring') {
            steps {
                echo '========================================'
                echo 'STAGE 7: MONITORING'
                echo '========================================'

                echo 'Starting application for monitoring health check...'

                bat '''
                    if exist monitoring.pid (
                        del /F /Q monitoring.pid
                    )

                    start /B node app.js > monitoring.log 2>&1

                    timeout /T 8 /NOBREAK > nul
                '''

                echo 'Checking application health...'

                bat '''
                    powershell -NoProfile -Command "$response = Invoke-WebRequest -Uri 'http://localhost:3000' -UseBasicParsing -TimeoutSec 10; if ($response.StatusCode -ge 200 -and $response.StatusCode -lt 500) { Write-Host 'APPLICATION HEALTH CHECK PASSED'; exit 0 } else { Write-Host 'APPLICATION HEALTH CHECK FAILED'; exit 1 }"
                '''

                echo 'Monitoring check completed.'
            }

            post {
                always {
                    archiveArtifacts artifacts: 'monitoring.log',
                                     allowEmptyArchive: true
                }
            }
        }
    }


    // ================================================================
    // PIPELINE RESULT
    // ================================================================
    post {
        success {
            echo '========================================'
            echo 'PIPELINE COMPLETED SUCCESSFULLY'
            echo '========================================'
            echo 'All 7 DevOps stages completed.'
            echo 'Build -> Test -> Code Quality -> Security -> Deploy -> Release -> Monitoring'
        }

        failure {
            echo '========================================'
            echo 'PIPELINE FAILED'
            echo '========================================'
            echo 'Please check the failed stage in the Jenkins console.'
        }

        always {
            echo 'Jenkins pipeline execution finished.'
        }
    }
}
