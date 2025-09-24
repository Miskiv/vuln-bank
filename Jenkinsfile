pipeline {
    agent none
    environment {
        DOCKERHUB_CREDENTIALS = credentials('DockerLogin')
    }
    stages {
        stage('Build') {
            agent any            
            steps {
                sh 'pip3 install --no-cache-dir -r requirements.txt'
            }
        }
        stage('Build Docker Image and Push to Docker Registry') {
            agent {
                docker {
                    image 'docker:dind'
                    args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
                }
	    }
	    environment {
                DOCKER_TLS_CERTDIR = ""
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
                    // Copy docker-compose.yml ke server
                    sh 'scp -i ${keyFile} -o StrictHostKeyChecking=no docker-compose.yml ubuntu@192.168.1.119:/home/ubuntu/docker-compose.yml'
                    // Login Docker di server
                    sh 'ssh -i ${keyFile} -o StrictHostKeyChecking=no ubuntu@192.168.1.119 "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"'
                    // Jalankan docker-compose up --build -d di server
                    sh 'ssh -i ${keyFile} -o StrictHostKeyChecking=no ubuntu@192.168.1.119 "cd /home/ubuntu && docker-compose up --build -d"'
                }
            }
        }
    }
}
