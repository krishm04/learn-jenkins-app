pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '19cd7f39-98ba-45ed-b17f-00943f442cee'
        NETLIFY_AUTH_TOKEN = credentials('NETLIFY_AUTH_TOKEN')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    apk add --no-cache bash git
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la build/
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
                    npm test -- --watchAll=false --ci --reporters=default --reporters=jest-junit
                '''
                junit 'junit.xml'
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
                script {
                    try {
                        sh '''
                            apk add --no-cache bash
                            npm install -g netlify-cli
                            echo "Deploying to Netlify..."
                            netlify deploy \
                                --dir=build \
                                --prod \
                                --auth=$NETLIFY_AUTH_TOKEN \
                                --site=$NETLIFY_SITE_ID \
                                --message "Deployed via Jenkins CI" \
                                --json > deploy-output.json
                            
                            # Extract deploy URL
                            DEPLOY_URL=$(jq -r '.deploy_url' deploy-output.json)
                            echo "Deployed to: ${DEPLOY_URL}"
                        '''
                    } catch (err) {
                        echo "Deployment failed: ${err}"
                        currentBuild.result = 'FAILURE'
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                if (fileExists('deploy-output.json')) {
                    DEPLOY_URL = sh(script: "jq -r '.deploy_url' deploy-output.json", returnStdout: true).trim()
                    emailext body: "Deployment successful!\n\nView at: ${DEPLOY_URL}", 
                            subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                            to: 'your-email@example.com'
                }
            }
            slackSend color: 'good', message: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
        failure {
            slackSend color: 'danger', message: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            emailext body: "Build failed!\n\nCheck console: ${env.BUILD_URL}", 
                    subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    to: 'your-email@example.com'
        }
        always {
            archiveArtifacts artifacts: 'build/**/*'
            junit '**/junit.xml'
            cleanWs()
        }
    }
}