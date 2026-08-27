전킨스 파일인데
들여쓰기 잘 맞는지 확인해줘
pipeline {
    agent {
        docker {
            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
            reuseNode true
        }
    }

    stages {
        stage('Build') {
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Test stage'
                sh '''
                    test -f build/index.html
                    JEST_JUNIT_OUTPUT_DIR=jest-results JEST_JUNIT_OUTPUT_NAME=junit.xml npm test
                    ls -la jest-results
                '''
            }
        }

        stage('E2E') {
            steps {
                sh '''
                    npm install serve
                    node_modules/.bin/serve -s build & sleep 10
                    npx playwright test
                    npx playwright test --reporter=html
                '''
            }
            stage('Deploy') {
            
            steps {
                sh '''
                    // 버전 꼭 맞추기!
                    npm install -g netlify-cli@20.1.1
                    netlify --version
                '''
            }
        }
        }
    }

    post {
        always {
            sh 'ls -la jest-results || echo MISSING'
            sh 'find . -name "junit.xml" -not -path "*/node_modules/*" 2>/dev/null'
            junit allowEmptyResults: true, testResults: '**/junit.xml'
        }
    }
}