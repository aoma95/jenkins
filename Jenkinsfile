@Library('slack-notifier')_
pipeline{
  environment{
    DOCKERHUB_CREDENTIALS=credentials('dockerhub_id')
    IMAGE_NAME = "tp"
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
    
    stage ('delete container test'){
      agent any
      steps{
        script{
          sh '''
          docker stop ${CONTAINER_NAME} && docker rm  ${CONTAINER_NAME}
          '''
        }
      }
    }
    stage ('push image in hub'){
      agent any
      steps{
        script{
          sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
          sh 'docker image tag ${IMAGE_NAME}:${IMAGE_TAG} aoma95/${IMAGE_NAME}:${IMAGE_TAG}'
          sh 'docker push aoma95/tp:${IMAGE_TAG}'
        }
      }
    }
    
    stage('Deploy app on EC2-cloud Production') {
            steps{
                withCredentials([sshUserPrivateKey(credentialsId: "ssh_prod", keyFileVariable: 'keyfile', usernameVariable: 'NUSER')]) {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        script{ 
                            sh'''
                                ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${IPPROD} -C \'docker rm -f static-webapp-prod-dan\'
                                ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${IPPROD} docker run -d --name static-webapp-prod-dan -p 8080:80 aoma95/${IMAGE_NAME}:${IMAGE_TAG}
                            ''' 
                        }
                    }
                }
            }
        }
     stage ('test online'){
      agent any
      steps{
        script{
          sh 'curl ${IPPROD}:8080 | grep -q "Contact"'
        }
      }
    }
  }
  post {
        always{
            script{
                slackNotifier currentBuild.result
            }
        }
    }
}
