pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'flask-app'
        REGISTRY_URL = 'us.icr.io/ibmproject/flask-form-app' // Update region if needed
        CLUSTER_NAME = 'minikube'
        DOCKER_CLI = 'docker:latest'
        DOCKER_CONFIG = '/var/jenkins_home/workspace/.docker'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/anagha-ananth/devops_flask_project.git'
            }
        }
        stage('Prepare Workspace') {
            steps {
                script {
                    sh '''
                    mkdir -p $DOCKER_CONFIG
                    chmod 700 $DOCKER_CONFIG
                    '''
                }
            }
        }
        stage('Lint Dockerfile') {
            steps {
                script {
                    sh '''
                    docker run --rm -i hadolint/hadolint < Dockerfile || true
                    '''
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.image(DOCKER_CLI).inside('--entrypoint=""') {
                        sh '''
                        docker --config $DOCKER_CONFIG build -t $REGISTRY_URL/$DOCKER_IMAGE .
                        '''
                    }
                }
            }
        }
        stage('Push Docker Image') {
        steps {
            withCredentials([string(credentialsId: 'ibm-cloud-api-key', variable: 'IBMCLOUD_API_KEY')]) {
                    script {
                        sh '''
                        # Define installation directory in user space
                        export IBMCLOUD_CLI_DIR=$HOME/ibmcloud-cli
                        export PATH=$IBMCLOUD_CLI_DIR:$PATH
        
                        # Check if IBM Cloud CLI is already installed
                        if ! command -v ibmcloud &> /dev/null
                        then
                            echo "IBM Cloud CLI not found. Installing..."
        
                            # Download IBM Cloud CLI
                            curl -fsSL https://clis.cloud.ibm.com/install/linux | tar -xz -C $HOME
        
                            # Move executable to user directory
                            mv $HOME/Bluemix_CLI/bin/ibmcloud $IBMCLOUD_CLI_DIR
        
                            # Ensure it's executable
                            chmod +x $IBMCLOUD_CLI_DIR/ibmcloud
                        else
                            echo "IBM Cloud CLI is already installed."
                        fi
                        
                        # Verify installation
                        ibmcloud --version
                        
                        # Log in to IBM Cloud with API key
                        ibmcloud login --apikey "$IBMCLOUD_API_KEY" -g Default
                        
                        # Log in to IBM Cloud Container Registry
                        ibmcloud cr login
                        
                        # Push the Docker image to IBM Cloud Registry
                        docker push $REGISTRY_URL/$DOCKER_IMAGE
                        '''
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh '''
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml
                    '''
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs.'
        }
    }
}
