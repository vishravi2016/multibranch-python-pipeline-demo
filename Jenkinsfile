pipeline {
    agent any

    options {
        timestamps()
        skipDefaultCheckout(true)
    }

    environment {
        VENV = 'venv'
    }

    stages {
        stage('Branch Information') {
            steps {
                echo "Building branch: ${env.BRANCH_NAME}"
                echo "Build number: ${env.BUILD_NUMBER}"
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Create Virtual Environment') {
            steps {
                bat '''
                    if exist venv rmdir /s /q venv
                    python -m venv venv
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                bat '''
                    call venv\\Scripts\\activate
                    python -m pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }
        stage('verify workspace'){
            steps{
                bat '''
                echo current directory
                cd
                echo.
                echo workspace files
                dir

                echo.
                echo app folder
                dir app

                echo.
                echo tests folder
                dir tests
                '''
            }
        }

        stage('Unit Tests') {
            steps {
                bat '''
                    call venv\\Scripts\\activate
                    
                    python -c "import sys; print(sys.path)"

                    set PYTHONPATH=%CD%

                    python -m pytest -v
                '''
            }
        }

        stage('Main Branch Verification') {
            when {
                branch 'main'
            }

            steps {
                echo 'Running production-ready verification for main branch'
            }
        }

        stage('Develop Branch Verification') {
            when {
                branch 'develop'
            }

            steps {
                echo 'Running integration verification for develop branch'
            }
        }

        stage('Feature Branch Verification') {
            when {
                expression {
                    env.BRANCH_NAME.startsWith('feature-')
                }
            }

            steps {
                echo "Running feature validation for ${env.BRANCH_NAME}"
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true,
                  testResults: 'reports/test-results.xml'

            archiveArtifacts artifacts: 'reports/**/*',
                             allowEmptyArchive: true
        }

        success {
            echo "${env.BRANCH_NAME} branch build completed successfully"
        }

        failure {
            echo "${env.BRANCH_NAME} branch build failed"
        }
    }
}
