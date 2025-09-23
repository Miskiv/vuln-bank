pipeline {
    agent none
    environment {
        DOCKERHUB_CREDENTIALS = credentials('DockerLogin')
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:lts-buster-slim'
                }               
            }
            steps {
                sh 'npm install'
            }
        }
        stage('Build Docker Image and Push to Docker Registry') {
            agent {
                docker {
                    image 'docker:dind'
                    args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                sh 'docker build -t miskiv/vuln-bank .'
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push miskiv/vuln-bank'
            }
        }
        stage('Deploy Docker Image') {
            agent {
                docker {
                    image 'kroniak/ssh-client'
                    args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: "DeploymentSSHKey", keyFileVariable: 'keyFile')]) {
                    sh 'ssh -i ${keyFile} -o StrictHostKeyChecking=no ubuntu@192.168.1.119 "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"'
                    sh 'ssh -i ${keyFile} -o StrictHostKeyChecking=no ubuntu@192.168.1.119 docker pull miskiv/vuln-bank'
                    sh 'ssh -i ${keyFile} -o StrictHostKeyChecking=no ubuntu@192.168.1.119 docker rm --force mongodb'
                    sh 'ssh -i ${keyFile} -o StrictHostKeyChecking=no ubuntu@192.168.1.119 docker run --detach --name mongodb -p 27017:27017 mongo:3'
                    sh 'ssh -i ${keyFile} -o StrictHostKeyChecking=no ubuntu@192.168.1.119 docker rm --force vuln-bank'
                    sh 'ssh -i ${keyFile} -o StrictHostKeyChecking=no ubuntu@192.168.1.119 docker run -it --detach --name vuln-bank --network host miskiv/vuln-bank'
                }
            }
        }
    }
}
