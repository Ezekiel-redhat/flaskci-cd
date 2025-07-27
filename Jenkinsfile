pipeline {

    agent any
 
    environment {

        IMAGE_NAME = "ezekiel/flaskapp"

        IMAGE_TAG = "latest"

    }
 
    stages {

        stage('Checkout') {

            steps {

                git url: 'https://github.com/Ezekiel-redhat/flaskci-cd.git', branch: 'main'

            }

        }
 
        stage('Install Dependencies') {

            steps {

                sh '''

                    echo "[+] Installing Python packages..."

                    pip3 install -r requirements.txt || true

                '''

            }

        }
 
        stage('Bandit SAST Scan') {

            steps {

                sh '''

                    echo "[+] Installing Bandit in user space..."

                    python3 -m pip install --user --upgrade pip

                    python3 -m pip install --user bandit

                    export PATH=$PATH:$HOME/.local/bin

                    echo "[+] Running Bandit..."

                    $HOME/.local/bin/bandit -r . -f html -o bandit-report.html || true

                '''

            }

        }
 
        stage('Build Docker Image') {

            steps {

                sh '''

                    echo "[+] Building Docker image ${IMAGE_NAME}:${IMAGE_TAG}..."

                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .

                '''

            }

        }
 
        stage('Trivy Image Scan') {

            steps {

                sh '''

                    echo "[+] Installing Trivy..."

                    sudo rpm -ivh https://github.com/aquasecurity/trivy/releases/latest/download/trivy_0.50.1_Linux-64bit.rpm || true

                    echo "[+] Running Trivy scan on Docker image..."

                    trivy image --exit-code 0 --format table --output trivy-report.txt ${IMAGE_NAME}:${IMAGE_TAG} || true

                '''

            }

        }
 
        stage('Run Container') {

            steps {

                sh '''

                    echo "[+] Running Docker container for testing..."

                    docker run -d -p 5000:5000 --name flask-container ${IMAGE_NAME}:${IMAGE_TAG}

                '''

            }

        }
 
        stage('Post-build Cleanup') {

            steps {

                sh '''

                    echo "[+] Stopping and cleaning up container..."

                    docker stop flask-container || true

                    docker rm flask-container || true

                '''

            }

        }

    }
 
    post {

        always {

            archiveArtifacts artifacts: 'bandit-report.html', fingerprint: true

            archiveArtifacts artifacts: 'trivy-report.txt', fingerprint: true

        }

    }

}

 
