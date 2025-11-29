pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup Python Env') {
            steps {
                sh '''
                    echo "Creating virtual environment..."
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
    }

    post {
        always {
            junit 'test-results.xml'
        }
    }
}

