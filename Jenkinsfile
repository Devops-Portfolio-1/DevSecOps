pipeline {
    agent any
    tools {
        nodejs 'Node.js 26.1.0'
    }

    stages {
        stage('VM Node Version') {
            steps {
                sh '''
                    node -v
                    npm -v
                '''
            }
        }
    }
}