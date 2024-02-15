pipeline {
    agent any
    environment {
            DOCKER_CREDS = credentials('a0e043f2-ca8c-4c10-9424-56e37c489094')
            }
    stages {
        stage('Build image') {
            steps {
                sh 'docker build -t ic-webapp projet-fils-rouge/.'
            }
        }
         stage('test image') {
            steps {
                sh 'docker rm -f test-ic-webapp || true'
                sh 'docker run --rm -p 80:8080 -d --name test-ic-webapp -e PGADMIN_URL=https://www.pgadmin.org/ -e ODOO_URL=https://www.odoo.com/fr_FR ic-webapp'
                sleep 5
                sh 'curl -I 192.168.1.50| grep 200'
                sh 'docker rm -f test-ic-webapp'
                sh 'rm -rf projet-fils-rouge'
            }
        }
         stage('release image') {
           
            steps {
                sh 'docker tag ic-webapp coolzie/ic-webapp:latest'
                sh 'docker login -u $DOCKER_CREDS_USR -p $DOCKER_CREDS_PSW'
                sh 'docker push coolzie/ic-webapp:latest'
            }
        }
        stage('deploy staging') {
            steps {
                 sshagent(credentials : ['sshkey']) {
                    sh 'ssh -o StrictHostKeyChecking=no coulibaly@192.168.1.100 docker rm -f test-ic-webapp || true'
                    sh 'ssh -o StrictHostKeyChecking=no coulibaly@192.168.1.100 docker login -u $DOCKER_CREDS_USR -p $DOCKER_CREDS_PSW'
                    sh 'ssh -o StrictHostKeyChecking=no coulibaly@192.168.1.100 docker run --rm -p 80:8080 -d --name test-ic-webapp -e PGADMIN_URL=https://www.pgadmin.org/ -e ODOO_URL=https://www.odoo.com/fr_FR coolzie/ic-webapp:latest'
                    sleep 5
                    sh 'ssh -o StrictHostKeyChecking=no coulibaly@192.168.1.100 docker ps | grep test-ic-webapp'
                }
            }
        }
        stage('test stagging') {
            steps {
                sh 'curl -I 192.168.1.100| grep 200'
            }
        }
        stage('deploy prod') {
            steps {
                 sshagent(credentials : ['sshkey']) {
                    sh 'ssh -o StrictHostKeyChecking=no coulibaly@192.168.1.100 docker rm -f prod-ic-webapp || true '
                    sh 'ssh -o StrictHostKeyChecking=no coulibaly@192.168.1.100 docker login -u $DOCKER_CREDS_USR -p $DOCKER_CREDS_PSW'
                    sh 'ssh -o StrictHostKeyChecking=no coulibaly@192.168.1.100 docker run --rm -p 81:8080 -d --name prod-ic-webapp -e PGADMIN_URL=https://www.pgadmin.org/ -e ODOO_URL=https://www.odoo.com/fr_FR coolzie/ic-webapp:latest'
                    sleep 5
                    sh 'ssh -o StrictHostKeyChecking=no coulibaly@192.168.1.100 docker ps | grep prod-ic-webapp'
                }
            }
        }
        stage('test prod') {
            steps {
                sh 'curl -I 192.168.1.100:81| grep 200'
            }
        }
    }
}
