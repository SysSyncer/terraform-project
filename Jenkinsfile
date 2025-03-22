pipeline {
    agent any
    environment {
        DOCKER_IMAGE_FRONTEND = "absurdguy/k8s-frontend:latest"  // Frontend Docker image name
        DOCKER_IMAGE_BACKEND = "absurdguy/k8s-backend:latest"  // Backend Docker image name
        CONTAINER_NAME_FRONTEND = "docker-k8s-frontend-app"  // Frontend container name
        CONTAINER_NAME_BACKEND = "docker-k8s-backend-app"  // Backend container name
        REGISTRY_CREDENTIALS = "dockerhub-project"  // Jenkins credentials ID for DockerHub
    }

    stages {
        stage('Checkout Code') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-project', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    // Clone both the backend and frontend repositories (if required)
                    git url: "https://$GIT_USER:$GIT_TOKEN@github.com/SysSyncer/terraform-project.git", branch: 'main'
                }
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                dir('backend') {
                    // Build Docker image for the backend
                    sh 'docker build -t $DOCKER_IMAGE_BACKEND .'
                }
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                dir('frontend') {
                    // Build Docker image for the frontend
                    sh 'docker build -t $DOCKER_IMAGE_FRONTEND .'
                }
            }
        }

        stage('Login to Docker Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-project', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    // Login to DockerHub registry
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Backend to Container Registry') {
            steps {
                // Push the backend Docker image to the registry
                sh 'docker push $DOCKER_IMAGE_BACKEND'
            }
        }

        stage('Push Frontend to Container Registry') {
            steps {
                // Push the frontend Docker image to the registry
                sh 'docker push $DOCKER_IMAGE_FRONTEND'
            }
        }

        stage('Stop & Remove Existing Containers') {
            steps {
                script {
                    // Stop and remove any existing containers for both frontend and backend
                    sh '''
                    if [ "$(docker ps -aq -f name=$CONTAINER_NAME_BACKEND)" ]; then
                        docker stop $CONTAINER_NAME_BACKEND || true
                        docker rm $CONTAINER_NAME_BACKEND || true
                    fi
                    if [ "$(docker ps -aq -f name=$CONTAINER_NAME_FRONTEND)" ]; then
                        docker stop $CONTAINER_NAME_FRONTEND || true
                        docker rm $CONTAINER_NAME_FRONTEND || true
                    fi
                    '''
                }
            }
        }

        stage('Run Backend Docker Container') {
            steps {
                // Run the backend Docker container
                sh 'docker run -d -p 5001:5000 --name $CONTAINER_NAME_BACKEND $DOCKER_IMAGE_BACKEND'
            }
        }

        stage('Run Frontend Docker Container') {
            steps {
                // Run the frontend Docker container
                sh 'docker run -d -p 5002:5000 --name $CONTAINER_NAME_FRONTEND $DOCKER_IMAGE_FRONTEND'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // You can deploy the backend and frontend containers using Kubernetes YAML files.
                // Assuming you are using kubectl to deploy with your k8s folder and your YAML files.

                script {
                    // Set up kubectl (make sure you have a kubeconfig file or the environment is set up)
                    sh '''
                    kubectl apply -f k8s/backend-deployment.yaml
                    kubectl apply -f k8s/frontend-deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Build, push, and container execution successful!"
        }
        failure {
            echo "Build or container execution failed."
        }
    }
}

