pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '19cd7f39-98ba-45ed-b17f-00943f442cee'
        NETLIFY_AUTH_TOKEN = credentials('NETLIFY_AUTH_TOKEN')
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
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
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    test -f build/index.html
                    npm test || true
                '''
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install -g netlify-cli
                    netlify deploy --dir=build --prod --auth=$NETLIFY_AUTH_TOKEN --site=$NETLIFY_SITE_ID --message "Deployed via Jenkins CI" --json
                '''
            }
        }
    }

    post {
        always {
            echo "Build and deploy completed"
        }
    }
}
