pipeline {
    agent any
    environment {
        REGISTRY = 'localhost:5000'
        IMAGE_NAME = 'webapp'
        DEPLOYMENT_FILE = '/home/admin13/webapp.yaml'
    }
    stages {
        stage('Preparation') {
            steps {
                checkout scm
                sh 'git rev-parse --short HEAD > .git/commit-id'
                script {
                    env.COMMIT_ID = readFile('.git/commit-id').trim()
                }
            }
        }
        stage('Compile') {
            steps {
                echo "Building the webapp"
                // Installer nvm et Node.js 16
                sh '''
                export NVM_DIR="$HOME/.nvm"
                [ -s "$NVM_DIR/nvm.sh" ] || curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
                . "$NVM_DIR/nvm.sh"
                nvm install 16
                nvm use 16
                node -v
                npm -v
                '''
                // Installer Angular CLI v16
                sh '''
                export NVM_DIR="$HOME/.nvm"
                . "$NVM_DIR/nvm.sh"
                npm install -g @angular/cli@16 || true
                ng version || true
                '''
                // Nettoyer et installer les dépendances
                sh '''
                export NVM_DIR="$HOME/.nvm"
                . "$NVM_DIR/nvm.sh"
                npm cache clean --force
                npm install
                ng build --prod
                '''
                echo "Build complete"
            }
        }
        stage('Build') {
            steps {
                echo "Building docker image ${REGISTRY}/${IMAGE_NAME}:${COMMIT_ID}"
                sh "docker build -t ${REGISTRY}/${IMAGE_NAME}:${COMMIT_ID} ."
                echo "Building complete"
                echo "Pushing docker image to registry"
                sh "docker push ${REGISTRY}/${IMAGE_NAME}:${COMMIT_ID}"
                echo "Push complete"
            }
        }
        stage('Deploy to Minikube') {
            steps {
                echo 'Starting Minikube if not running...'
                sh 'minikube status || minikube start'
                echo 'Deploying to Minikube...'
                sh "minikube image load ${REGISTRY}/${IMAGE_NAME}:${COMMIT_ID}"
                sh "kubectl set image deployment/webapp webapp=${REGISTRY}/${IMAGE_NAME}:${COMMIT_ID} --namespace=k8s-fleetman"
                sh "kubectl apply -f ${DEPLOYMENT_FILE}"
                echo 'Deployment complete'
            }
        }
    }
    post {
        always {
            echo 'Cleaning up...'
            sh 'npm cache clean --force || true'
            sh 'docker system prune -f || true'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
