def commit_id

pipeline {
    agent any

    stages {
        stage('Preparation') {
            steps {
                checkout scm
                sh 'git rev-parse --short HEAD > .git/commit-id'
                script {
                    commit_id = readFile('.git/commit-id').trim()
                }
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying to Minikube...'
                sh "sed -i 's/commit_id/${commit_id}/g' ./manifests/webapp.yaml"
                sh "kubectl get all"
                sh "kubectl apply -f ./manifests/"
                echo 'Deployment complete'
            }
        }
    }
}
