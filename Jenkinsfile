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
                echo "Printing everything on docker container after building"
                ls -la
                '''
            }
        }
    }
}