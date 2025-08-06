pipeline {
    environment {
        IMAGE_NAME = "alpinehelloworld"
        IMAGE_TAG= "latest"
        STAGING = "easylab-gitlab-staging"
        PRODUCTION = "easylab-gitlab-prod"
    }
    agent none
    stages {
        stage('Build Image') {
            agent any
            steps {
                script {
                    sh 'docker build -t danipro237/$IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }

    stage('Run Container base on Build Image') {
        agent any
        steps {
            script {
                sh '''docker run --name $IMAGE_NAME -d -p 80:5000 -e PORT=5000 danipro237/$IMAGE_NAME:$IMAGE_TAG 
                      sleep 5s
                '''
            }
        }
    }

    stage('Test image') {
        agent any
        steps {
            script {
                sh ''' curl http://localhost | grep -q "Hello world!"
                '''
            }
        }
    }

    stage('Destroy container') {
        agent any
        steps {
            script {
                sh ''' docker stop $IMAGE_NAME && docker rm $IMAGE_NAME '''
            }
        }
    }

    stage('Push Image in staging and deploy') {
        when{
            expression { GIT_BRANCH == 'origin/jenkins' }
        }
        agent any
        environment{
            HEROKU_API_KEY = credentials('heroku_api_key')
        }
        steps {
            script {
                sh '''
                        heroku container:login
                        heroku create $STAGING || echo "project alredy exist"
                        docker container:push -a $STAGING web
                        heroku container:release -a $STAGING web
                '''
            }
        }
    }

    stage('Push Image in production and deploy') {
        when{
            expression { GIT_BRANCH == 'origin/jenkins' }
        }
        agent any
        environment{
            HEROKU_API_KEY = credentials('heroku_api_key')
        }
        steps {
            script {
                sh '''
                        heroku container:login
                        heroku create $PRODUCTION || echo "project alredy exist"
                        docker container:push -a $PRODUCTION web
                        heroku container:release -a $PRODUCTION web
                '''
            }
        }
    }
  }
}
