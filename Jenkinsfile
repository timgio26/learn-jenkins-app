pipeline {
    agent any

    stages {
        /*
        stage('Build') {
            agent{
                docker{
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
        */

        stage('Test') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                # test -f build/index.html
                npm test
                '''
            }
        }

        stage('E2E') {
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.52.0-noble'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm i serve
                # & running in background
                node_modules/.bin/serve -s build &
                # tunggu server ready
                sleep 10
                npm playwright test
                '''
            }
        }
    }

    post{
        always{
            junit 'test-results/junit.xml'
        }
    }
}
