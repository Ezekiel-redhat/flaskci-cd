pipeline {
    agent any
 
    environment {
        IMAGE_NAME = 'my-flask-app'
        IMAGE_TAG = 'latest'
    }
 
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/Ezekiel-redhat/flask-ci-cd.git'
            }
        }
 
        stage('Install Dependencies') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }
 
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }
 
        stage('Scan with Trivy') {
            steps {
                sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL $IMAGE_NAME:$IMAGE_TAG'
            }
        }
 
        stage('Push to Docker Registry') {
            when {
                expression { return env.BRANCH_NAME == 'main' }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker tag $IMAGE_NAME:$IMAGE_TAG $DOCKER_USER/$IMAGE_NAME:$IMAGE_TAG
                    docker push $DOCKER_USER/$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }
    }
 
    post {
        failure {
            mail to: 'devops-team@cisco.com',
                 subject: "Jenkins Pipeline Failed: ${env.JOB_NAME}",
                 body: "Job ${env.JOB_NAME} failed. Check Jenkins console: ${env.BUILD_URL}"
        }
    }
}
