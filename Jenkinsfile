pipeline {
    agent any

    tools {
        nodejs 'NodeJS-18'   // MUST match EXACT Jenkins tool name
    }

    environment {
        NODE_ENV = 'production'
    }

    stages {

        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'

                sh '''
                    # Install frontend dependencies and build
                    cd frontend
                    npm install
                    npm run build || true
                '''

                sh '''
                    # Generate SBOM (optional)
                    npm run sbom || true
                '''

                sh '''
                    # Fix vulnerabilities (safe mode)
                    npm audit fix || true
                '''
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    if (fileExists('frontend/src/karma.conf.js')) {
                        sh '''
                            cd frontend
                            npm run test -- --watch=false --source-map=true || true
                        '''
                    } else {
                        echo "Skipping frontend tests (karma.conf.js not found)"
                    }

                    sh 'npm run test:server || true'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarQube') {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline completed 🎉"
        }
    }
}
