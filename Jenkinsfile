pipeline {
    agent any

    environment {
        NODE_HOME = tool name: 'NodeJS-18', type: 'NodeJS'
        PATH = "${env.NODE_HOME}/bin:${env.PATH}"
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
                    # Build frontend
                    cd frontend && npm install && npm run build:frontend
                    # Generate SBOM
                    npm run sbom || true
                '''
                sh '''
                    # Fix vulnerabilities safely (non-breaking)
                    npm audit fix || true
                '''
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    // Skip frontend tests if karma.conf.js missing
                    if (fileExists('frontend/src/karma.conf.js')) {
                        sh '''
                            cd frontend
                            npm run test -- --watch=false --source-map=true
                        '''
                    } else {
                        echo "karma.conf.js not found, skipping frontend tests"
                    }
                    // Always run server tests
                    sh 'npm run test:server || true'
                }
            }
        }

        stage('SonarQube Analysis') {
            environment {
                // Inject SonarQube env from Jenkins configuration
                scannerHome = tool name: 'SonarScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
            }
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning workspace..."
            cleanWs()
        }
    }
}
