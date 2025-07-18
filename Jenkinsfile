pipeline {
    agent any
 
    environment {
        IMAGE_NAME = 'my-flask-app'
        IMAGE_TAG = 'latest'
    }
 
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Ezekiel-redhat/flaskci-cd.git'
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
        sh '''
            # Fail the build if HIGH or CRITICAL vulnerabilities are found
            trivy image --exit-code 0 --severity HIGH,CRITICAL --format table -o trivy-report.txt $IMAGE_NAME:$IMAGE_TAG
        '''
        archiveArtifacts artifacts: 'trivy-report.txt', fingerprint: true
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
