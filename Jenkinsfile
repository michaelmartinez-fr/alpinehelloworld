pipeline {
  environment {
    IMAGE_NAME = "alpinehelloworld"
    IMAGE_TAG = "0.2"
    STAGING = "easzy-staging"
    PRODUCTION = "eazy-production"
    DOCKER_USERNAME = "micmartin"
  }
  // default use none
  agent none
  stages {
      stage('Build image') {
        agent any
        steps {
          script {
            sh 'docker build -t ${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} .'
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
              docker run -d -p 80:5000 -e PORT=5000 --name ${IMAGE_NAME} ${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}
              sleep 5
            '''
          }
        }
      }
      stage('Test image') {
        agent any
        steps {
          script {
            // multilines
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
            // multilines
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
        agent any
        environment {
            DOCKERHUB_PAT = credentials('dockhub_pat_devo')
        }
        steps {
          script {
            sh '''
              echo ${DOCKERHUB_PAT} | docker login -u ${DOCKER_USERNAME} --password-stdin
              docker push ${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}
            '''
          }
        }
      }  

 }
}