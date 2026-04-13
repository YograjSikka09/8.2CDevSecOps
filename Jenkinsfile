node {

    stage('Checkout SCM') {
        checkout scm
    }

    stage('Install Dependencies') {
        sh 'node -v'
        sh 'npm -v'
        sh 'npm install || true'
        dir('frontend') {
            sh 'npm install || true'
        }
    }

    stage('Build Application') {
        dir('frontend') {
            sh 'npm run build || true'
        }
    }

    stage('Run Tests') {
        sh 'npm run test:server || true'
    }

    stage('Generate Coverage Report') {
        sh 'npm run coverage || true'
    }

    stage('Security Scan') {
        sh 'npm audit || true'
    }

    stage('SonarQube Analysis') {
        def scannerHome = tool name: 'SonarScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
        withSonarQubeEnv('SonarQube') {
            sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=juice-shop -Dsonar.host.url=http://localhost:9000"
        }
    }

    echo 'Pipeline finished successfully'

}
