pipeline {
    agent any
    environment{
        FULL_IMAGE = "909927813182.dkr.ecr.ap-southeast-1.amazonaws.com/nodejs-random-color:latest"
        TASK_DEFINITION =""
        NEW_TASK_DEFINITION=""
        NEW_TASK_INFO=""
        NEW_REVISION=""
        TASK_FAMILY="nodejs-task-definition"
    }
    stages {
        stage('Build') {
            steps {
                sh 'docker build -t nodejs-random-color:latest .'
            }
        }
        stage('Upload image to ECR') {
            steps {
                sh 'aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 909927813182.dkr.ecr.ap-southeast-1.amazonaws.com'
                sh 'docker tag nodejs-random-color:latest 909927813182.dkr.ecr.ap-southeast-1.amazonaws.com/nodejs-random-color:latest'
                sh 'docker push 909927813182.dkr.ecr.ap-southeast-1.amazonaws.com/nodejs-random-color:latest'
            }
        }
        
        stage('Update task definition and force deploy ECS service') {
            steps {
                sh '''
                    # Fetch the current task definition
                    TASK_DEFINITION=$(aws ecs describe-task-definition --task-definition ${TASK_FAMILY} --region "ap-southeast-1")

                    # Remove unnecessary fields and update the image
                    NEW_TASK_DEFINITION=$(echo $TASK_DEFINITION | jq --arg IMAGE "${FULL_IMAGE}" '
                        .taskDefinition
                        | .containerDefinitions[0].image = $IMAGE
                        | del(
                            .taskDefinitionArn,
                            .revision,
                            .status,
                            .requiresAttributes,
                            .compatibilities,
                            .registeredAt,
                            .registeredBy
                          )
                    ')

                    # Register the new task definition
                    NEW_TASK_INFO=$(aws ecs register-task-definition --region "ap-southeast-1" --cli-input-json "$NEW_TASK_DEFINITION")

                    # Extract the full ARN and the revision
                    NEW_TASK_ARN=$(echo $NEW_TASK_INFO | jq -r '.taskDefinition.taskDefinitionArn')
                    NEW_REVISION=$(echo $NEW_TASK_INFO | jq -r '.taskDefinition.revision')

                    echo "New revision: $NEW_REVISION"
                    echo "New task ARN: $NEW_TASK_ARN"

                    # Update the ECS service with the full ARN
                    aws ecs update-service \
                      --cluster udemy-devops-ecs-cluster \
                      --service nodejs-service \
                      --task-definition ${NEW_TASK_ARN} \
                      --force-new-deployment
                '''
            }
        }
    }
}
