pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '50f4f389-8434-4f38-a0e7-f09195c9c552'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
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
                    echo 'test v2'
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage("Run Tests") {
            parallel {
              stage('Unit Tests') {
                  agent {
                      docker {
                          image 'node:18-alpine'
                          reuseNode true
                      }
                  }
                  steps {
                      echo 'Test Stage'
                      sh '''
                          if [ ! -f build/index.html ]; then
                              echo 'Build output missing'
                              exit 1
                          fi

                          npm run test
                      '''
                  }
                  post {
                    always {
                        junit 'jest-results/junit.xml'
                    }
                  }
              }

              stage('E2E') {
                  agent {
                      docker {
                          image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                          reuseNode true
                      }
                  }
                  steps {
                      sh '''
                          npm install serve
                          node_modules/.bin/serve -s build &
                          sleep 10
                          npx playwright test --reporter=html
                      '''
                  }

                  post {
                    always {
                        publishHTML([
                            allowMissing: false,
                            alwaysLinkToLastBuild: false,
                            icon: '',
                            keepAll: false,
                            reportDir: 'playwright-report',
                            reportFiles: 'index.html',
                            reportName: 'Playwright HTML Report',
                            reportTitles: 'Playwright Report',
                            useWrapperFileDirectly: true
                        ])
                    }
                  }
              }
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
                npm i netlify-cli@20.1.1
                node_modules/.bin/netlify --version
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
    }
}
