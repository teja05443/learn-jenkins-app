pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'b82e32ec-c325-4c48-89ed-8f6b49323ef9'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {

        stage('AWS Build')
        {
            agent
            {
                docker
                {
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"  
                }
            }
            steps
            {
                sh'''
                    aws --version
                '''
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
                    echo "Small Change"
                    ls -la
                    npm --version
                    node --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Test Parallel') {
            parallel {
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
                            npm test
                        '''
                    }
                }
                stage('E2E') {
                    agent {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            serve -s build &
                            sleep 10
                            npx playwright test
                        '''
                    }
                }
            }
        }

        stage('Deploy Staging') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "STAGING URL NEEDS TO BE SET"
            }
            steps {
                sh '''
                    netlify --version
                    echo "Deploying to staging. SITE ID : $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                    npx playwright test
                '''
            }
        }

        stage('Approval') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you wish to deploy to production', ok: 'Yes, I am sure!'
                }
            }
        }

        stage('Deploy Prod') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://dreamy-otter-4dd3f5.netlify.app'
            }
            steps {
                sh '''
                    node --version
                    netlify --version 
                    echo "Deploying to production. SITE ID : $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                    npx playwright test
                '''
            }
        }
    }
}
