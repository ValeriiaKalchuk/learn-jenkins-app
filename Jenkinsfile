pipeline {
	agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        APP_NAME = 'my-jenkinsapp'
        AWS_DOCKER_REGISTRY = '070785328482.dkr.ecr.us-east-1.amazonaws.com'
        AWS_DEFAULT_REGION = 'us-east-1'
        AWS_ECS_CLUSTER_PROD = 'LearningJenkins-Cluster-Prod'
        AWS_ECS_SERVICE_PROD = 'LearningJenkins-Service-Prod'
        AWS_ECS_TASK_D_PROD = 'LearningJenkins-TaskDefinition-Prod'
    }

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
        stage('Build Docker image') {
			agent {
				docker {
					image 'my-aws-cli'
					reuseNode true
					args "-u root -v /run/docker.sock:/run/docker.sock --entrypoint=''"
                }
            }
            steps {
				withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
					sh '''
						docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION .
						aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
						docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
					'''
				}
            }
        }
		stage('Deploy to AWS') {
			agent {
				docker {
					image 'my-aws-cli'
					reuseNode true
					args "--entrypoint=''"
                }
            }
            steps {
				withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
					sh '''
						aws --version
						sed -i "s/#APP_VERSION#/$REACT_APP_VERSION/g" aws/task-definition-prod.json
						LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
						aws ecs update-service --cluster $AWS_ECS_CLUSTER_PROD --service $AWS_ECS_SERVICE_PROD --task-definition $AWS_ECS_TASK_D_PROD:$LATEST_TD_REVISION
						aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER_PROD --services $AWS_ECS_SERVICE_PROD
					'''
				}
			}
        }
    }
}
