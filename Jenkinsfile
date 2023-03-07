#!/usr/bin/env groovy
pipeline{
        
    agent any
        
    parameters {
      string(defaultValue: "1.0", description: 'IMAGE TAG', name: 'IMAGE_TAG')  
      string(defaultValue: "demo-cluster", description: 'AWS ECS Cluster', name: 'AWS_ECS_CLUSTER')
      string(defaultValue: "ap-southeast-1", description: 'AWS ECS Region', name: 'AWS_ECR_REGION')  
      string(defaultValue: "bmi-ui-svc-ext", description: 'AWS ECS Service', name: 'AWS_ECS_SERVICE')
      string(defaultValue: "uat", description: 'AWS ECS Service Environment', name: 'AWS_ECS_SERVICE_ENVIRONMENT')
      string(defaultValue: "bmp_ui-task-definition", description: 'AWS ECS Task Definition', name: 'AWS_ECS_TASK_DEFINITION')  
      string(defaultValue: "", description: 'AWS ECS Repository URL', name: 'AWS_ECR_REPO_URL')  
    }

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
        AWS_ECS_TASK_DEFINITION_PATH = './task_def.json'
    }
    
   
    stages {            
        stage('Deploy in ECS') {
            steps {
                withCredentials([string(credentialsId: 'AWS_REPOSITORY_URL_SECRET', variable: 'AWS_ECR_URL')]) {
                    withAWS(region: "${AWS_ECR_REGION}", credentials: 'silapakarn') {
                        script {
                            def image = "${AWS_ECR_REPO_URL}/${AWS_ECS_SERVICE}:${IMAGE_TAG}"
                            sh("cat ./Infra/${AWS_ECS_SERVICE_ENVIRONMENT}.json | jq '.containerDefinitions[].image = \"$image\"' > ./Infra/${AWS_ECS_SERVICE_ENVIRONMENT}.json")
                            sh("/usr/local/bin/aws ecs register-task-definition --region ${AWS_ECR_REGION} --cli-input-json file://Infra/${AWS_ECS_SERVICE_ENVIRONMENT}.json")
                            def taskRevision = sh(script: "/usr/local/bin/aws ecs describe-task-definition --task-definition ${AWS_ECS_TASK_DEFINITION} | jq -r '.taskDefinition.revision'", returnStdout: true)
                            sh("/usr/local/bin/aws ecs update-service --force-new-deployment --cluster ${AWS_ECS_CLUSTER} --service ${AWS_ECS_SERVICE} --task-definition ${AWS_ECS_TASK_DEFINITION}:${taskRevision}")
                            
                        }
                    }
                }
            }
        }
        
      
        
    }
}
