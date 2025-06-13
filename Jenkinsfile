pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'harshj2003/cicd-jenkins-ansible:latest'
    }

    stages {

        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/Harshwardhanjadhav0/CI-CD-pipeline-using-Jenkins-Ansible-Docker.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin'
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                sh 'docker push $DOCKER_IMAGE'
            }
        }

        stage('Deploy via Ansible') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'target-ec2-key',
                        keyFileVariable: 'SSH_KEY'
                    ),
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "Running Ansible Playbook..."
                        /usr/bin/ansible-playbook -i inventory.ini playbook.yaml \
                          --private-key $SSH_KEY \
                          -e "ansible_user=ubuntu ansible_ssh_private_key_file=$SSH_KEY docker_username=$DOCKER_USERNAME docker_password=$DOCKER_PASSWORD"
                    '''
                }
            }
        }
    }
}

