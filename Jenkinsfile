pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = "e0989720-fe2d-4eae-b113-91c76dd1aa7a"
        NETLIFY_AUTH_TOKEN = credentials("netlify-token")
    }
    
    stages{
        
        stage("Without Docker"){
            steps{
                echo 'Without Docker'
                echo 'Printing everything on Jenkins agent'
                sh 'ls -la'
            }
        }
        
        stage("With Docker"){
            
            agent{
               docker{
                   image 'node:18-alpine'
                   reuseNode true
               } 
            }
            
            steps{
                echo 'With Docker'
                sh '''
                echo "Printing everything on docker container before building"
                ls -la
                node --version
                npm --version
                npm ci
                npm run build
                echo "Printing everything on docker container after building"
                ls -la
                '''
            }
        }

        stage("Tests"){

            parallel{

                stage("Unit Test"){

                    agent {
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps{
                        sh'''
                        echo 'Test Stage'
                        test -f "build/index.html"
                        echo $?
                        npm test
                        '''
                    }

                    post{

                        always{
                            junit "jest-results/junit.xml"
                        }
                    }
                }

                stage("E2E Playwright Test"){

                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    steps{
                        sh'''
                        npm install serve
                        node_modules/.bin/serve -s build &
                        sleep 10
                        npx playwright test --reporter=html
                        '''
                    }

                    
                    post{

                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])

                        }
                    }
                }
            }

        }

        stage("Deploy Staging"){


            agent {
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps{
                sh'''
                npm install netlify-cli@20.1.1 node-jq
                node_modules/.bin/netlify --version
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --json > deploy_staging.json
                '''

                script{
                    env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy_staging.json", returnStdout: true)
                }
            }

        }

        stage("Staging E2E"){


            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment{
                CI_ENVIRONMENT_URL = "$env.STAGING_URL"
            }

            steps{
                sh'''
                echo "URL_FOR_STAGING_SERVER: $CI_ENVIRONMENT_URL"
                npx playwright test --reporter=html
                '''
            }

                    
            post{

                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Staging E2E Report', reportTitles: '', useWrapperFileDirectly: true])

                }
            }
        }

        stage("Approval"){

            steps{

                timeout(time: 5, unit: 'MINUTES') {
                    input message: 'Do you want to approve the build deployment?', ok: 'Yes, I am sure.'
                }

            }

        }

        stage("Deploy Prod"){


            agent {
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps{
                sh'''
                npm install netlify-cli@20.1.1
                node_modules/.bin/netlify --version
                echo "Deployment ID for the site: $NETLIFY_SITE_ID"
                echo "NETLIFY_AUTH_TOKEN: $NETLIFY_AUTH_TOKEN"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }

        }

        stage("Prod E2E"){


            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment{
                CI_ENVIRONMENT_URL = "https://effortless-elf-7a39a0.netlify.app"
            }

            steps{
                sh'''
                echo "Naal Nachna.........."
                echo "URL_FOR_PRODUCTION_SERVER: $CI_ENVIRONMENT_URL"
                npx playwright test --reporter=html
                '''
            }

                    
            post{

                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Prod E2E Report', reportTitles: '', useWrapperFileDirectly: true])

                }
            }
        }
          
    }


}