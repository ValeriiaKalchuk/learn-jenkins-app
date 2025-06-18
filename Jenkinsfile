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
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo 'Test stage - positive test'
                sh '''
                    echo "Checking if index.html exists in the build folder"
                    test -f build/index.html
                    echo "Running unit tests"
                    npm test
                '''
                sh '''
                    echo Test stage - negative test
                    test -f build/x.html
                '''

            }
        }
    }

    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}
