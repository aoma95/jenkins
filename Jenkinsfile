pipeline{
  environment{
    IMAGE_NAME = "website"
    IPPROD = "10.13.9.68"
    CONTAINER_NAME = "theweb"
    IMAGE_TAG = "${BUILD_TAG}"
    STAGING = "ynov-dan-staging"
    PRODUCTION = "ynov-dan-prod"
  }
  agent any
  
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
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
        script{
          sh '''
            docker stop ${CONTAINER_NAME} && docker rm  ${CONTAINER_NAME}
            docker run -d --name ${CONTAINER_NAME} -p 8081:80 ${IMAGE_NAME}:${IMAGE_TAG}
          '''
        }
      }
      }
    }
    
    stage ('Test image'){
      agent any
      steps{
        script{
        //catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          sh '''
            curl localhost:8081 | grep -q "Contact"
          '''
        }
      }

      }
    
    stage ('deploy'){
      agent any
      steps{
        script{
          sh '''
          docker stop ${CONTAINER_NAME} && docker rm  ${CONTAINER_NAME}
          '''
        }
      }
    }
    
    stage('Deploy app on EC2-cloud Production') {
            steps{
                withCredentials([sshUserPrivateKey(credentialsId: "ssh_prod", keyFileVariable: 'keyfile', usernameVariable: 'NUSER')]) {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        script{ 
                            sh'''
                                ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${IPPROD} -C \'sudo docker rm -f static-webapp-prod\'
                                ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${IPPROD} -C \'sudo docker run -d --name static-webapp-prod  -e PORT=80 -p 80:80 sadofrazer/alpinehelloworld\'
                            '''
                        }
                    }
                }
            }
        }
  }
}
