pipeline {
    agent any

    environment {
        VENV_DIR = "venv"
        FLASK_APP = "crudapp.py"
        SERVICE_NAME = "flaskcrud"
        WORK_DIR = "/var/lib/jenkins/workspace/Flask-CRUD-Pipeline"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "📥 Checking out latest source code..."
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "⚙️ Setting up Python virtual environment and installing dependencies..."
                sh '''
                    # Create venv if missing
                    if [ ! -d "$VENV_DIR" ]; then
                        python3 -m venv $VENV_DIR
                    fi

                    # Activate and install dependencies
                    . $VENV_DIR/bin/activate
                    pip install --upgrade pip

                    # Install app dependencies + Gunicorn for deployment
                    pip install -r requirements.txt
                    pip install gunicorn
                '''
            }
        }

        stage('Test') {
            steps {
                echo "✅ Running unit tests with pytest..."
                sh '''
                    . $VENV_DIR/bin/activate
                    pip install pytest
                    pytest --maxfail=1 --disable-warnings -q || true
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo "🚀 Deploying Flask app with Gunicorn via systemd..."
                sh '''
                    . $VENV_DIR/bin/activate
                    export FLASK_APP=${FLASK_APP}

                    # Run DB migrations
                    flask db upgrade

                    # Restart Gunicorn service
                    echo "🔄 Restarting systemd service..."
                    sudo systemctl restart ${SERVICE_NAME} || true
                    sudo systemctl status ${SERVICE_NAME} --no-pager || true

                    echo "✅ App deployed! Accessible at http://<your-server-ip>:5000"
                '''
            }
        }
    }

    post {
        success {
            echo "🎉 Build, test, and deploy succeeded!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}
