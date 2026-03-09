pipeline {
  environment {
    IMAGE_NAME = "alpinehelloworld"
    IMAGE_TAG = "latest"
    STAGING = "easzy-staging"
    PRODUCTION = "eazy-production"
  }
  # default
  agent none
  stages {
      stage('Build image') {
        agent any
        steps {
          script {
            sh 'docker build -t micmartin/${IMAGE_NAME}:${IMAGE_TAG} .'
          }
        }
      }
      stage('Run container') {
        agent any
        steps {
          script {
            # multilines
            sh '''
              docker run -d -p 80:5000 -e PORT=5000 --name ${IMAGE_NAME} micmartin/${IMAGE_NAME}:${IMAGE_TAG}
              sleep 5
            '''
          }
        }
      }
      stage('Test image') {
        agent any
        steps {
          script {
            # multilines
            sh '''
              curl http://localhost | grep -q "Hello world!"
            '''
          }
        }
      }
      stage('clean container') {
        agent any
        steps {
          script {
            # multilines
            sh '''
              docker rm -f ${IMAGE_NAME}
            '''
          }
        }
      }
      stage('Test image') {
        agent any
        steps {
          script {
            # multilines
            sh '''
              curl http://localhost | grep -q "Hello world!"
            '''
          }
        }
      }
      stage('Push image in staging and deploy') {
        when {
              expression { GIT_BRANCH == 'origin/master' }
             }
        agent any
        environment {
            DOCKERHUB_PAT = credentials('dockerhub_pat')
        }
        steps {
          script {
            sh '''
              docker login -u micmartin -p ${DOCKERHUB_PAT}
              docker push micmartin/${IMAGE_NAME}
            '''
          }
        }
      }  

 }
}
