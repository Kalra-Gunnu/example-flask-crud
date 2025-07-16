pipeline {
    agent any

    environment {
        VENV = 'venv'
    }

    stages {
        stage('Build') {
            steps {
                echo '⚙️  Setting up Python virtual environment and installing dependencies...'
                sh '''
                    python3 -m venv $VENV
                    . $VENV/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                echo '✅ Running unit tests with pytest...'
                sh '''
                    . $VENV/bin/activate
                    if [ -d "tests" ]; then
                        pip install pytest
                        pytest --maxfail=1 --disable-warnings -q
                    else
                        echo "No tests directory found — skipping tests."
                    fi
                '''
            }
        }

        stage('Deploy') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                echo '🚀 Deploying Flask app to staging environment...'
                sh '''
                    pkill -f crudapp.py || true

                    nohup $VENV/bin/python crudapp.py > flask_app.log 2>&1 &

                    echo "✅ App deployed. Access it at http://<your-server-ip>:5000"
                '''
            }
        }
    }

    post {
        always {
            echo "📦 Pipeline finished with status: ${currentBuild.currentResult}"
        }
        success {
            echo "🎉 Build, test, and deploy succeeded!"
        }
        failure {
            echo "🚨 Build or test failed!"
        }
    }
}
