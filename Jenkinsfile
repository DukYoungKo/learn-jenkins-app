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