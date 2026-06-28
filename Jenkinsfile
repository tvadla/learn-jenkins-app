pipeline {
    agent any

    stages {
        // this is a comment
        /*
        line of comments
        */
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
        stage('Test'){
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            
            steps{
                echo "test stage"
                sh '''
                    #this is a comment
                    test -f build/index.html
                    npm test
                '''
            }
        }
    }
    post {
        always{
            junit 'test-results/junit.xml'
        }
    }
}
