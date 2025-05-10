pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "kubeirshad/train-schedule:${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/irshadkabir/cicd-pipeline-train-schedule-autodeploy.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $DOCKER_IMAGE .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-prod1', variable: 'KUBECONFIG'),
                                 usernamePassword(credentialsId: 'aws-access-credentials', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                        # Set up AWS credentials
                        export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                        export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                        export AWS_DEFAULT_REGION=us-east-1  # Make sure the region is set

                        # Set up Kubeconfig for Kubernetes
                        export KUBECONFIG=$KUBECONFIG

                        # Apply the Kubernetes configurations
                        kubectl apply -f train-schedule-kube.yml

                        # Set the new Docker image in Kubernetes
                        kubectl set image deployment/train-schedule train-schedule=$DOCKER_IMAGE --record
                    '''
                }
            }
        }
    }
}

