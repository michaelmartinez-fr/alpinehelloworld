pipeline {
  environment {
    IMAGE_NAME = "alpinehelloworld"
    IMAGE_TAG = "0.2"
    DOCKERHUB_USERNAME = "micmartin"
    // here or in an stage's environment {}
    // MUST set a credential of type "Secret text" with ID=dockhub_pat_devo and secret=XXXXX
    // no other explicit set of the credential of pipeline definition
    // DOCKERHUB_PAT = credentials('dockhub_pat_devo')
    STAGING = "easzy-staging"
    PRODUCTION = "eazy-production"
  }
  // default use none
  agent none
  stages {
      stage('Build image') {
        agent any
        steps {
          script {
            sh 'docker build -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} .'
          }
        }
      }
      stage('Run container') {
        agent any
        steps {
          script {
            // multilines
            sh '''
              echo "Cleaning existing container if exist"
              docker ps -a | grep -i $IMAGE_NAME && docker rm -f $IMAGE_NAME
              docker run -d -p 80:5000 -e PORT=5000 --name ${IMAGE_NAME} ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}
              sleep 5
            '''
          }
        }
      }
      stage('Test image') {
        agent any
        steps {
          script {
            sh '''
              curl http://localhost | grep -q "Hello world!"
            '''
          }
        }
      }
      stage('Clean container') {
        agent any
        steps {
          script {
            sh '''
              docker rm -f ${IMAGE_NAME}
            '''
          }
        }
      }
      stage('Push image to DockerHub') {
        when {
              expression { GIT_BRANCH == 'origin/master' }
             }
        environment {
              DOCKERHUB_PAT = credentials('dockhub_pat_devo')
            }
        agent any
        steps {
          script {
            sh '''
              echo ${DOCKERHUB_PAT} | docker login -u ${DOCKERHUB_USERNAME} --password-stdin
              docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}
            '''
          }
        }
      }  

 }
}
