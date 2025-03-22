pipeline {
    agent any
    environment {
    MINIKUBE_HOME = '/home/student/.minikube'  // Path to Minikube if it's not default
    KUBEVERSION = '1.20.0'  // Optional: Define Kubernetes version if needed
        DOCKER_IMAGE_FRONTEND = "frontend:latest"  // Frontend Docker image name
        DOCKER_IMAGE_BACKEND = "backend:latest"  // Backend Docker image name
        CONTAINER_NAME_FRONTEND = "docker-k8s-frontend-app"  // Frontend container name
        CONTAINER_NAME_BACKEND = "docker-k8s-backend-app"  // Backend container name
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Checkout the code from the repository
                git url: "https://github.com/SysSyncer/terraform-project.git", branch: 'main'
            }
        }

        stage('Set Minikube Docker Environment') {
            steps {
                script {
                    // Set up the Minikube Docker environment for Jenkins
                    sh '''
                    eval $(minikube docker-env)
                    '''
                }k
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                dir('backend') {
                    // Build the Docker image for the backend using Minikube's Docker daemon
                    sh 'docker build -t $DOCKER_IMAGE_BACKEND .'
                }
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                dir('frontend') {
                    // Build the Docker image for the frontend using Minikube's Docker daemon
                    sh 'docker build -t $DOCKER_IMAGE_FRONTEND .'
                }
            }
        }

        stage('Load Images into Minikube') {
            steps {
                // Load the Docker images into Minikube's Docker daemon
                sh '''
                minikube image load $DOCKER_IMAGE_BACKEND
                minikube image load $DOCKER_IMAGE_FRONTEND
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Apply Kubernetes deployment and service YAMLs using kubectl
                    sh '''
                    kubectl delete -f k8s/backend-deployment.yaml
                    kubectl delete -f k8s/frontend-deployment.yaml
                    kubectl delete -f k8s/service.yaml
                    kubectl delete -f k8s/configmap.yaml

                    kubectl apply -f k8s/backend-deployment.yaml
                    kubectl apply -f k8s/frontend-deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    kubectl apply -f k8s/configmap.yaml
                    '''
                }
            }
        }

        stage('Verify Pods and Services') {
            steps {
                // Verify the status of the pods and services in Kubernetes
                sh '''
                kubectl get pods
                kubectl get svc
                '''
            }
        }

        stage('Test Frontend Service') {
            steps {
                // Test the frontend service to ensure it's accessible via Minikube
                sh '''
                FRONTEND_URL=$(minikube service frontend-service --url)
                echo "Frontend Service URL: $FRONTEND_URL"
                '''
            }
        }

        stage('Test Backend Service') {
            steps {
                // Test the backend service by sending a curl request
                sh '''
                kubectl run test-pod --image=alpine --restart=Never -it -- sh
                apk add curl  # Install curl if not available
                curl http://backend-service:5000/products
                '''
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
