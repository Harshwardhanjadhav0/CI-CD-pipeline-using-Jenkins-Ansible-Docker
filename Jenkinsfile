pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'harshj2003/cicd-jenkins-ansible:latest'
    }

    stages {

        stage('Clone Repo') {
            steps {
                // Cloning from GitHub
                git branch: 'main', url: 'https://github.com/Harshwardhanjadhav0/CI-CD-pipeline-using-Jenkins-Ansible-Docker.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                // Build image using Dockerfile in current directory
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Login to DockerHub') {
            steps {
                // Pull credentials from Jenkins credentials store
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    // Login to DockerHub
                    sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin'
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                // Push Docker image to your DockerHub repository
                sh 'docker push $DOCKER_IMAGE'
            }
        }

        stage('Deploy via Ansible') {
            steps {
                // Ensure Ansible is available on the Jenkins host
                sh 'which ansible-playbook || echo "ERROR: Ansible not found!"'

                // Inject DockerHub credentials as extra-vars into Ansible playbook
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''
                        /usr/bin/ansible-playbook -i inventory.ini playbook.yaml \
                        --extra-vars "docker_username=$DOCKER_USERNAME docker_password=$DOCKER_PASSWORD"
                    '''
                }
            }
        }
    }
}

