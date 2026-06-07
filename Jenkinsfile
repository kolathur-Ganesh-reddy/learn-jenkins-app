pipeline {
    agent any

    environment {
        NETLIFY_PROJECT_ID = 'ef63c21b-460b-4c0b-9f5a-eb64e780e573'
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

        stage('deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm install netlify-cli
                node_modules/.bin/netlify --version
                echo 'deploying to netlify... Project ID: $NETLIFY_PROJECT_ID'
                node_modules/.bin/netlify status
                '''
            }
        }
    }
}