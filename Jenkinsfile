pipeline {
  environment {
    IMAGE_NAME = "alpinehelloworld"
    IMAGE_TAG = "0.4"
    DOCKERHUB_USERNAME = "micmartin"
    // here or in an stage's environment {}
    // MUST set a credential of type "Secret text" with ID=dockhub_pat_devo and secret=XXXXX
    // no other explicit set of the credential of pipeline definition
    // DOCKERHUB_PAT = credentials('dockhub_pat_devo')

    // param relative to easylabs/easylabs tool to deploy on VM (franela/dind)
    STAGING = "eazy-staging"
    // PRODUCTION = "eazy-production"
    // STG_API_ENDPOINT = "${PARAM_STG_API_ENDPOINT}"        /* Mettre le couple IP:PORT de votre API eazylabs, exemple 100.25.147.76:1993 */
    // STG_APP_ENDPOINT = "${PARAM_STG_APP_ENDPOINT}"        /* Mettre le couple IP:PORT votre application en staging, exemple 100.25.147.76:8000 */

    // got from test Open Port 1993: For API use -1993., for APP use -80
    PROD_API_ENDPOINT =  "ip10-0-2-5-d6o1rmm57ed000fbtukg-1993.direct.docker.labs.eazytraining.fr"  /* "${PARAM_PROD_API_ENDPOINT}" */      /* Mettre le couple IP:PORT de votre API eazylabs, 100.25.147.76:1993 */
    PROD_APP_ENDPOINT =  "ip10-0-2-5-d6o1rmm57ed000fbtukg-80.direct.docker.labs.eazytraining.fr"   /* "${PARAM_PROD_APP_ENDPOINT}" */
    INTERNAL_PORT = "5000"
    APP_EXPOSED_PORT = 80 /* "${PARAM_PORT_EXPOSED}" */            /*80 par défaut*/
    EXTERNAL_PORT = 80   /* "${PARAM_PORT_EXPOSED}" */
    CONTAINER_IMAGE = "${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"
    
    // param of curl call to eazylabs (?)
    APP_NAME = "${IMAGE_NAME}" /* "${PARAM_APP_NAME}" */
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
      stage('PROD - Deploy app') {
       when {
             expression { GIT_BRANCH == 'origin/master' } /* correct to master */
            }
       agent any
       steps {
          script {
            sh """
              # APP_NAME = IMAGE_NAME, container_image: micmartin/alpinehelloworld:0.3, external port 80, internal port 5000
              echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json
              # https NEEDED prod used to create prod-alpinehelloworld
              curl -v -X POST https://${PROD_API_ENDPOINT}/prod -H 'Content-Type: application/json' --data-binary @data.json 2>&1 | grep 200
            """
          }
       }
     }

 }
}
