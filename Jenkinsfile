pipeline {
    agent any
    
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

        stage("Test"){

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
        }

        stage("E2E Playwright Test"){

            agent{
                docker{
                    image 'docker pull mcr.microsoft.com/playwright:v1.58.0-noble'
                    reuseNode true
                }
            }

            steps{
                sh'''
                npm install serve
                node_modules/.bin/serve -s build
                npx playwright test
                '''
                
            }
        }
    }

    post{

        always{
            junit "jest-results/junit.xml"
        }
    }
}