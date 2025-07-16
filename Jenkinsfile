pipeline {
    agent any

    environment {
        VENV = 'venv'
        FLASK_APP = 'crudapp.py'
    }

    stages {

        stage('Checkout') {
            steps {
                echo '📥 Checking out latest source code...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo '⚙️  Setting up Python virtual environment and installing dependencies...'
                sh '''
                    # Create venv if not exists
                    if [ ! -d "$VENV" ]; then
                        python3 -m venv $VENV
                    fi

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
                    # Install pytest if not already present
                    pip install pytest

                    if [ -d "tests" ]; then
                        pytest --maxfail=1 --disable-warnings -q
                    else
                        echo "⚠️ No tests found. Skipping test stage."
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
                    . $VENV/bin/activate
                    export FLASK_APP=crudapp.py

                    # --- Database Setup ---
                    if [ ! -d "migrations" ]; then
                        flask db init
                    fi
                    flask db migrate -m "Auto migration from Jenkins deploy"
                    flask db upgrade

                    # Kill any running instance
                    pkill -f "flask run" || true
                    pkill -f crudapp.py || true

                    # Start Flask app accessible externally
                    nohup flask run --host=0.0.0.0 --port=5000 > flask_app.log 2>&1 &

                    echo "✅ App deployed! Access it at http://<your-server-ip>:5000"
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
            echo "🚨 Build or test failed! Check logs."
        }
    }
}
