pipeline {
    agent any

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
                echo 'Test Stage'
                sh '''
                if [ ! -f build/index.html ]; then
                  echo 'Build output missing'
                  exit 1
                fi

                npm run test
                '''
                
              }
          }
    }
}
