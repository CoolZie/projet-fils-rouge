pipeline {
    agent any
    environment {
            DOCKER_CREDS = credentials('a0e043f2-ca8c-4c10-9424-56e37c489094')
            GIT_COMMIT_SHORT = sh(
                    script: "printf \$(git rev-parse --short ${GIT_COMMIT})",
                    returnStdout: true
            )
            DEPLOY_HOST = 192.168.1.100
            IMAGE_NAME = 'ic-webapp'
            }
    stages {
        stage('Build image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }
         stage('test image') {
            steps {
                sh 'docker rm -f test-$IMAGE_NAME || true'
                sh 'docker run --rm -p 80:8080 -d --name test-$IMAGE_NAME -e PGADMIN_URL=https://www.pgadmin.org/ -e ODOO_URL=https://www.odoo.com/fr_FR $IMAGE_NAME'
                sleep 5
                sh 'curl -I $DEPLOY_HOST| grep 200'
                sh 'docker rm -f test-$IMAGE_NAME'
                sh 'rm -rf projet-fils-rouge'
            }
        }
        stage('deploy & test review') {
           when {
                branch '*/review'
            }
           steps {
              sh 'docker build -t $GIT_COMMIT_SHORT-$IMAGE_NAME .'
              sh 'docker run --rm -p 88:8080 -d --name review-$GIT_COMMIT_SHORT-$IMAGE_NAME -e PGADMIN_URL=https://www.pgadmin.org/ -e ODOO_URL=https://www.odoo.com/fr_FR $GIT_COMMIT_SHORT-$IMAGE_NAME'
              sleep 5
              sh 'curl -I $DEPLOY_HOST:88 | grep 200'
              sh 'docker rm -f review-$GIT_COMMIT_SHORT-$IMAGE_NAME'
              sh 'docker image rm -f $GIT_COMMIT_SHORT-$IMAGE_NAME'
              sh 'rm -rf projet-fils-rouge'
               
           }
        }
        
         stage('release image') {
           
            steps {
                sh 'docker tag $IMAGE_NAME coolzie/$IMAGE_NAME:latest'
                sh 'docker login -u $DOCKER_CREDS_USR -p $DOCKER_CREDS_PSW'
                sh 'docker push coolzie/$IMAGE_NAME:latest'
            }
        }
 
        stage('deploy staging') {
            steps {
                 sshagent(credentials : ['sshkey']) {
                    sh 'ssh -o StrictHostKeyChecking=no coulibaly@$DEPLOY_HOST docker rm -f test-$IMAGE_NAME || true'
                    sh 'ssh -o StrictHostKeyChecking=no coulibaly@$DEPLOY_HOST docker login -u $DOCKER_CREDS_USR -p $DOCKER_CREDS_PSW'
                    sh 'ssh -o StrictHostKeyChecking=no coulibaly@$DEPLOY_HOST docker run --rm -p 80:8080 -d --name test-$IMAGE_NAME -e PGADMIN_URL=https://www.pgadmin.org/ -e ODOO_URL=https://www.odoo.com/fr_FR coolzie/$IMAGE_NAME:latest'
                    sleep 5
                    sh 'ssh -o StrictHostKeyChecking=no coulibaly@$DEPLOY_HOST docker ps | grep test-$IMAGE_NAME'
                }
            }
        }
        stage('test stagging') {
            steps {
                sh 'curl -I $DEPLOY_HOST| grep 200'
            }
        }
        stage('deploy prod') {
            steps {
                 sshagent(credentials : ['sshkey']) {
                    sh 'ssh -o StrictHostKeyChecking=no coulibaly@$DEPLOY_HOST docker rm -f prod-$IMAGE_NAME || true '
                    sh 'ssh -o StrictHostKeyChecking=no coulibaly@$DEPLOY_HOST docker login -u $DOCKER_CREDS_USR -p $DOCKER_CREDS_PSW'
                    sh 'ssh -o StrictHostKeyChecking=no coulibaly@$DEPLOY_HOST docker run --rm -p 81:8080 -d --name prod-$IMAGE_NAME -e PGADMIN_URL=https://www.pgadmin.org/ -e ODOO_URL=https://www.odoo.com/fr_FR coolzie/$IMAGE_NAME:latest'
                    sleep 5
                    sh 'ssh -o StrictHostKeyChecking=no coulibaly@$DEPLOY_HOST docker ps | grep prod-$IMAGE_NAME'
                }
            }
        }
        stage('test prod') {
            steps {
                sh 'curl -I $DEPLOY_HOST:81| grep 200'
            }
        }
    }
}
