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
        stage('Test stage') {
            steps {
                agent {
                    docker {
                        image 'node:18-alpine'
                        reuseNode true
                }
            }
                echo 'Test stage'
                sh '''
                    echo "Checking if index.html exists in the build folder"
                    test -f build/index.html
                    echo "Running unit tests"
                    npm test
                '''
            }
        }
    }
}
