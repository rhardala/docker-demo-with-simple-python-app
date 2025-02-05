/* import shared library */
@Library('shard-library')_


pipeline {
     environment {
       IMAGE_NAME = "my-python-app"
       IMAGE_TAG = "1.0.1"
       CONTAINER_PORT = "8080"
       HOST_PORT = "80"
       HOST_IP = "172.17.0.1"
       STAGING = "choco1992-staging"
       PRODUCTION = "choco1992-production"
     }
     agent none
     stages {
         stage('Build image') {
             agent any
             steps {
                script {
                  sh 'docker build -t choco1992/$IMAGE_NAME:$IMAGE_TAG .'
                }
             }
        }
        stage('Run container based on builded image') {
            agent any
            steps {
               script {
                 sh '''
                    docker run --name $IMAGE_NAME -d -p ${HOST_PORT}:${CONTAINER_PORT}  choco1992/$IMAGE_NAME:$IMAGE_TAG
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
                    curl http://${HOST_IP} | grep -q "Hello World"
                '''
              }
           }
      }
      stage('Clean Container') {
          agent any
          steps {
             script {
               sh '''
                 docker stop $IMAGE_NAME
                 docker rm $IMAGE_NAME
               '''
             }
          }
     }
     stage('Push image in staging and deploy it') {
       when {
              expression { GIT_BRANCH == 'origin/master' }
            }
      agent any
      environment {
          HEROKU_API_KEY = credentials('heroku_api_key')
      }  
      steps {
          script {
            sh '''
              heroku container:login
              heroku create $STAGING || echo "project already exist"
              heroku container:push -a $STAGING web
              heroku container:release -a $STAGING web
            '''
          }
        }
     }
     stage('Push image in production and deploy it') {
       when {
              expression { GIT_BRANCH == 'origin/master' }
            }
      agent any
      environment {
          HEROKU_API_KEY = credentials('heroku_api_key')
      }  
      steps {
          script {
            sh '''
              heroku container:login
              heroku create $PRODUCTION || echo "project already exist"
              heroku container:push -a $PRODUCTION web
              heroku container:release -a $PRODUCTION web
            '''
          }
        }
     }
  }
  post {
       always {
       script {
         slackNotifier currentBuild.result
     }
    }  
    }
}
