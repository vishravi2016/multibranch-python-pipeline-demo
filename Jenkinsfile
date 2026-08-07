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
                deleteDir()
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

                    echo Cleaning previous reports

                    rmdir /s /q reports 2>nul
                    mkdir reports

                    echo Running unit tests
                    set PYTHONPATH=%CD%          
                   
                    python -m pytest -v --junitxml=reports\\test-results.xml            

                    
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

        stage('production readiness check') {
            when {
                branch 'main'
            }

            steps {
                echo 'checking production readiness'
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
            junit testResults: 'reports/test-results.xml',
                    allowEmptyResults:false

            archiveArtifacts artifacts: 'reports/**/*',
                             allowEmptyArchive: true
        }

        success {
            echo "${env.BRANCH_NAME} branch build completed successfully"
        }

        failure {
            echo "${env.BRANCH_NAME} branch build failed"
        }
        unstable{
             echo "${env.BRANCH_NAME} branch build is unstable"
        }
    }
}
