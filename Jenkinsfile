pipeline{
  environment{
    IMAGE_NAME = "website"
    CONTAINER_NAME = "theweb"
    IMAGE_TAG = "${BUILD_TAG}"
    STAGING = "ynov-dan-staging"
    PRODUCTION = "ynov-dan-prod"
  }
  agent none
  
  stages{
    
    stage ('Build Stage'){
      agent any
      steps{
        script{
          sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
        }
      }
    } 
    stage ('Run Container based on builded image'){
      agent any
      steps{
        script{
          sh '''
            docker run -d --name ${CONTAINER_NAME} -p 8081:80 ${IMAGE_NAME}:${IMAGE_TAG}
          '''
        }
      }
    }
    
    stage ('Test image'){
      agent any
      steps{
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          sh '''
            curl localhost:8081 | grep -q "Codzfntact"
          '''
        }
      }
    }
    
    stage ('Delete Container'){
      agent any
      steps{
        script{
          sh '''
            docker stop ${CONTAINER_NAME}
            docker rm ${CONTAINER_NAME}
          '''
        }
      }
    }
  }
}
