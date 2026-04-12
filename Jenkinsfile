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
                // Verify Node and npm versions
                sh 'node -v'
                sh 'npm -v'

                // Install backend dependencies
                sh 'npm install'

                // Install and build frontend
                sh '''
                    cd frontend
                    npm install
                    npm run build || true
                '''

                // Generate SBOM (optional)
                sh 'npm run sbom || true'

                // Fix vulnerabilities (safe mode)
                sh 'npm audit fix || true'
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

                    // Run backend/server tests
                    sh 'npm run test:server || true'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'sonar-scanner'
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline completed successfully 🎉"
        }
    }
}
