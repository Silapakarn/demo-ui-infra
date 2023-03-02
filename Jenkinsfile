#!/usr/bin/env groovy
pipeline{
        
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
        timeout(time: 1, unit: 'HOURS')
        timestamps()
    }
    
    tools {
        maven 'maven 3.9.0'
    }
    
     environment {
        AWS_ECR_REGION = 'ap-southeast-1'
        AWS_ECS_CLUSTER = 'my-first-cluster'
        AWS_ECS_TASK_DEFINITION = 'bmp_ui-task-definition'
        AWS_ECS_TASK_DEFINITION_INPUT_PATH = './Infra/uat.json'
        AWS_ECS_TASK_DEFINITION_PATH = './task_def.json'
        AWS_ECS_SERVICE = 'demo-counter-app-service'
    }
    
   
    stages {            
        stage('Deploy in ECS') {
            steps {
                withCredentials([string(credentialsId: 'AWS_REPOSITORY_URL_SECRET', variable: 'AWS_ECR_URL')]) {
                    withAWS(region: "${AWS_ECR_REGION}", credentials: 'silapakarn') {
                        script {
                            def image = "${AWS_ECR_URL}:${IMAGE_TAG}"
                            sh 'pwd'
                            sh 'ls'
                            sh("cat ${AWS_ECS_TASK_DEFINITION_INPUT_PATH} | jq '.containerDefinitions[].image = \"$image\"' > ${AWS_ECS_TASK_DEFINITION_PATH}")
                            sh("/usr/local/bin/aws ecs register-task-definition --region ${AWS_ECR_REGION} --cli-input-json file://${AWS_ECS_TASK_DEFINITION_PATH}")
                            def taskRevision = sh(script: "/usr/local/bin/aws ecs describe-task-definition --task-definition ${AWS_ECS_TASK_DEFINITION} | jq -r '.taskDefinition.revision'", returnStdout: true)
                            sh("/usr/local/bin/aws ecs update-service --force-new-deployment --cluster ${AWS_ECS_CLUSTER} --service ${AWS_ECS_SERVICE} --task-definition ${AWS_ECS_TASK_DEFINITION}:${taskRevision}")
                            
                        }
                    }
                }
            }
        }
        
      
        
    }
}