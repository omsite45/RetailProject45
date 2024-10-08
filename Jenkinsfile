
pipeline {
    agent any

    environment {
        LABS = credentials('jupyter_lab_creds')  // Jenkins credentials ID for secure deployment
    }

    stages {
        stage('Build') {
            steps {
                echo 'Starting the build process...'

                // Check if pipenv is installed, otherwise install it
                sh '''
                    if ! command -v pipenv &> /dev/null
                    then
                        echo "pipenv is not installed, installing it..."
                        python3 -m venv $HOME/venv  # Changed to use $HOME for compatibility
                        source $HOME/venv/bin/activate  # Activating the virtual environment
                        pip install --upgrade pip
                        pip install pipenv
                    else
                        echo "pipenv is already installed"
                    fi

                '''

                // Clean up old virtual environment if it exists
                echo 'Cleaning up old virtual environment...'
                sh '/bitnami/jenkins/home/.local/bin/pipenv --rm || exit 0'

                // Install dependencies using pipenv
                echo 'Installing dependencies using pipenv...'
                sh '/bitnami/jenkins/home/.local/bin/pipenv install'

                echo 'Build completed successfully!'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests using pytest...'
                // Run pytest in the pipenv environment
                sh '/bitnami/jenkins/home/.local/bin/pipenv run pytest'
            }
        }

        stage('Package') {
            steps {
                echo 'Packaging project into a zip file...'

                // Package the project into a zip file and handle errors
                sh '''
                    if [ -d . ]; then
                        zip -r retailproject.zip .
                        echo "Packaging completed successfully!"
                    else
                        echo "Error: Project directory does not exist!"
                        exit 1
                    fi
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying project to the remote server...'

                // Securely transfer files using sshpass and SCP
                sh '''
                    sshpass -p "$LABS_PSW" scp -o StrictHostKeyChecking=no -r ."$LABS_USR"@g02.itversity.com:/home/itv014498/retailproject
                    if [ $? -eq 0 ]; then
                        echo "Deployment successful!"
                    else
                        echo "Error during deployment!"
                        exit 1
                    fi
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed, please check the logs!'
        }
    }
}
