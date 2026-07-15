pipeline {
    agent any
    environment {
        HARBOR_REG = 'harbor-node1.com/myapp'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }
    stages {
        stage('Checkout') {
            steps { git branch: 'main', url: 'https://github.com/peakyblinder0509/myapp.git' }
        }
        stage('SonarQube Scan') {
    steps {
        script {
            def scannerHome = tool 'SonarScanner'
            withSonarQubeEnv('SonarQube') {
                sh "${scannerHome}/bin/sonar-scanner"
            }
        }
    }
}
        stage('Build Frontend Image') {
            steps { sh "docker build -t ${HARBOR_REG}/frontend:${IMAGE_TAG} ./frontend" }
        }
        stage('Build Backend Image') {
            steps { sh "docker build -t ${HARBOR_REG}/backend:${IMAGE_TAG} ./backend" }
        }
        stage('Push to Harbor') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'test-harbor', usernameVariable: 'U', passwordVariable: 'P')]) {
                    sh "echo \$P | docker login harbor-node1.com -u 'robot\$jenkins' --password-stdin"
                    sh "docker push ${HARBOR_REG}/frontend:${IMAGE_TAG}"
                    sh "docker push ${HARBOR_REG}/backend:${IMAGE_TAG}"
                }
            }
        }
        stage('Deploy to K8s') {
            steps {
                sh "kubectl set image deployment/frontend-deployment frontend=${HARBOR_REG}/frontend:${IMAGE_TAG}"
                sh "kubectl set image deployment/backend-deployment backend=${HARBOR_REG}/backend:${IMAGE_TAG}"
            }
        }
    }
    post {
        success { echo 'Deployment success!' }
        failure { echo 'Pipeline failed, check logs' }
    }
}
