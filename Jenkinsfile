pipeline {
    agent any
    
    environment{
        NETLIFY_SITE_ID='a761b83c-7047-4ae2-a606-abf09182ca55'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {

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

        stage('run tests'){
            parallel{
                stage('unit test') {
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
                    post{
                        always{
                            junit 'jest-results/junit.xml'
                            // publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                            // junit 'test-results-e2e/junit.xml'
                        }
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
                        npx playwright test --reporter=html
                        '''
                    }
                    post{
                        always{
                            // junit 'jest-results/junit.xml'
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright LOCAL Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        // stage('deploy staging'){
        //     agent{
        //         docker{
        //             image 'node:18-alpine'
        //             reuseNode true
        //         }
        //     }
        //     steps{
        //         sh '''
        //             npm install netlify-cli@20.1.1
        //             npm i node-jq
        //             node_modules/.bin/netlify --version
        //             echo "deploying to staging $NETLIFY_SITE_ID"
        //             node_modules/.bin/netlify status
        //             node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
        //             node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json
        //         '''
        //         script{
        //             env.STAGING_URL = sh(script:"node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json",returnStdout:true)
        //         }
        //     }
        // }

        stage('deplpoy staging + E2E') {
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.52.0-noble'
                    reuseNode true
                }
            }
            // environment{
            //     // CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            //     CI_ENVIRONMENT_URL = 'please replace'
            // }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    npm i node-jq
                    node_modules/.bin/netlify --version
                    echo "deploying to staging $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                    
                    #important to be no space
                    CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json)
                    
                    sleep 10
                    npx playwright test --reporter=html
                '''
            }
            post{
                always{
                    // junit 'jest-results/junit.xml'
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright staging E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('approval') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // some block
                    input message: 'ready to deploy?', ok: 'Yes deploy'
                }
            }
        }

        // stage('deploy production'){
        //     agent{
        //         docker{
        //             image 'node:18-alpine'
        //             reuseNode true
        //         }
        //     }
        //     steps{
        //         sh '''
        //             npm install netlify-cli@20.1.1
        //             node_modules/.bin/netlify --version
        //             echo "deploying to $NETLIFY_SITE_ID"
        //             node_modules/.bin/netlify status
        //             node_modules/.bin/netlify deploy --dir=build --prod
        //         '''
        //     }
        // }

        stage('deploy Prod + E2E') {
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.52.0-noble'
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = 'https://glittering-gelato-fa6864.netlify.app'
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "deploying to $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                    sleep 30
                    npx playwright test --reporter=html
                '''
            }
            post{
                always{
                    // junit 'jest-results/junit.xml'
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }

    // post{
    //     always{
    //         junit 'jest-results/junit.xml'
    //         publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
    //         // junit 'test-results-e2e/junit.xml'
    //     }
    // }
}
