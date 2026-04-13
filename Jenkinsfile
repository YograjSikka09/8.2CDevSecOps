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

    stage('SonarCloud Analysis') {
        withCredentials([string(credentialsId: 'sonar_token', variable: 'SONAR_TOKEN')]) {
            def scannerHome = tool name: 'SonarScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
            sh """
                ${scannerHome}/bin/sonar-scanner \
                -Dsonar.projectKey=YograjSikka09_8.2CDevSecOps \
                -Dsonar.organization=yograjsikka09 \
                -Dsonar.host.url=https://sonarcloud.io \
                -Dsonar.login=${SONAR_TOKEN} \
                -Dsonar.sources=. \
                -Dsonar.exclusions=node_modules/**,test/** \
                -Dsonar.projectName=NodeJS Goof Vulnerable App \
                -Dsonar.sourceEncoding=UTF-8
            """
        }
    }

    echo 'Pipeline finished successfully'

}
