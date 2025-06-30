pipeline {
	agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        AWS_DEFAULT_REGION = 'us-east-1'
    }

    stages {
		stage('AWS') {
			agent {
				docker {
					image 'amazon/aws-cli'
					reuseNode true
					args "-u root --entrypoint=''"
                }
            }
            steps {
				withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
					sh '''
						aws --version
						yum install jq -y
						LATEST_TD_REVISION = $(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod. | jq '.taskDefinition.revision')
						aws ecs update-service --cluster LearningJenkins-Cluster-Prod --service LearningJenkins-Service-Prod --task-definition LearningJenkins-TaskDefinition-Prod:$LATEST_TD_REVISION
					'''
				}
			}
        }
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
    }
}
