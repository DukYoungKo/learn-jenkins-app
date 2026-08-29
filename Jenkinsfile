pipeline {
    agent {
        docker {
            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
            reuseNode true
        }
    }

    environment {
        NETLIFY_SITE_ID = 'c31c2be1-71cb-4c52-9421-7306043e861f'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Build') {
            steps {
                sh '''
                    echo '트리거 테스트 중..'
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
                    echo "프로젝트 배포중.. 사이트아이디 : $NETLIFY_SITE_ID"
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    # 버전 꼭 맞추기!
                    npm install netlify-cli@20.1.1
                    ./node_modules/.bin/netlify --version
                    ./node_modules/.bin/netlify status
                    ./node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }

        stage('Prod E2E') {
            environment {
                CI_ENVIRONMENT_URL = 'https://taupe-kitsune-0f2650.netlify.app'
            }

            steps {
                sh '''
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