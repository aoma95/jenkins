pipeline{
  environment{
    IMAGE_NAME = "website"
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
            docker run -d --name ${CONTAINER_NAME} -e PORT=8080 -p 8080:80 ${IMAGE_NAME}:${IMAGE_TAG}
          '''
        }
      }
    }
    
    stage ('Test image'){
      agent any
      steps{
        script{
          sh '''
            curl localhost:8080 | grep -q "Hello world!"
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
