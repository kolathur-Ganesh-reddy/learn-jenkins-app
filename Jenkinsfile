pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'e33dd9e6-4f84-4863-a578-b5e22442a7a4'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }
    stages {
        stage('build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm ci
                npm run build
                '''
            }
        }

        stage('Tests') {
            parallel {
                stage('unit test') {
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
                            image 'mcr.microsoft.com/playwright:v1.60.0-noble'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                        npm install serve
                        node_modules/.bin/serve -s build &
                        sleep 10
                        npx playwright test --reporter=line
                        '''
                    }  
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }    
                }
            }
        }

        stage('Approval') {
            steps {
                input cancel: 'cancel the deploy', message: 'i wish you ', ok: 'ok ready to deploy'
            }
        }

        stage('deploy') {
            agent {
                docker {
                    image 'node:18'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm install netlify-cli
                node_modules/.bin/netlify --version
                echo 'deploying to netlify... Site ID: $NETLIFY_SITE_ID'
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
    }
}