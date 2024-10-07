pipeline{
    agent any 
    
    environment {
        NETLIFY_SITE_ID = '32ce7a58-a435-49b1-b945-7390945d49c8'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages{
        stage('Docker'){
            steps{
                sh 'docker build -t my-playwright .'
            }
        }
        stage('Build'){
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            steps{
                sh'''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage('Run Tests'){
            parallel{
                stage('Test'){
                    agent {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            echo Test stage
                            ls -la
                            test -f build/index.html
                            npm test
                        '''
                    }
                }
                stage('E2E'){
                    agent {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            serve -s build & 
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                }
            }
        }
        stage('Deploy Staging'){
            agent {
                 docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'STAGING_URL'
            }
            steps{
                sh'''
                    netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                    npx playwright test  --reporter=html
                '''
            }
            post {
                always{
                    junit 'jest-results/junit.xml'
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'Staging.html', reportName: 'Staging HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
        stage('Approval'){
            steps{
                timeout(time: 15, unit: 'MINUTES'){
                        input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                }
            }
        }
        stage('Deploy Prod'){
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://storied-genie-beb5a1.netlify.app'
            }
            steps{
                sh'''
                    netlify --version
                    echo "Deploying to production. Site ID : $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                '''
            }
            post {
                always{
                    junit 'jest-results/junit.xml'
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'Production.html', reportName: 'Production HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        } 
    }
}