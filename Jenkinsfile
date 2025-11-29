pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "sunsjz/python-ci-cd-demo"
    }

    stages {

        // ------------------- CI 部分 --------------------
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup Python Env') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    . venv/bin/activate
		    export PYTHONPATH=/var/jenkins_home/workspace/python-ci/
                    pytest -q --junitxml=test-results.xml
                '''
            }
        }

        // ------------------- 构建 Docker 镜像 --------------------
        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building Docker image..."
                    docker build -t $DOCKER_IMAGE:$BUILD_NUMBER .
                    docker tag $DOCKER_IMAGE:$BUILD_NUMBER $DOCKER_IMAGE:latest
                '''
            }
        }

        // ------------------- 推送 Docker Hub --------------------
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD'
                )]) {
                    sh '''
                        echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
                        docker push $DOCKER_IMAGE:$BUILD_NUMBER
                        docker push $DOCKER_IMAGE:latest
                    '''
                }
            }
        }

        // ------------------- 本地自动部署（A） --------------------
        stage('Deploy Locally') {
            steps {
                sh '''
                    echo "Stopping old container..."
                    docker rm -f python-ci-app || true

                    echo "Starting new container..."
                    docker run -d --name python-ci-app -p 8000:8000 $DOCKER_IMAGE:latest
                '''
            }
        }
    }

    post {
        always {
            junit 'test-results.xml'
        }
    }
}


