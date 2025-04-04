pipeline {
    agent any

    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal-py'
        RESOURCE_GROUP = 'rg-jenkins'
        APP_SERVICE_NAME = 'jenkinspythonfastapi123876857'
        PYTHON_EXE = 'C:\\Users\\rajsi\\AppData\\Local\\Programs\\Python\\Python313\\python.exe' 
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Raj-Singh71/python-azure-app.git'
            }
        }

        stage('Setup Python Environment') {
            steps {
                bat "${PYTHON_EXE} -m venv venv"
                bat ".\\venv\\Scripts\\pip install fastapi uvicorn gunicorn"
            }
        }

        stage('Package App') {
            steps {
                

                bat 'powershell Compress-Archive -Path main.py, requirements.txt -DestinationPath pythonapp.zip -Force'
            }
        }

        stage('Deploy to Azure') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat "az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID"
                    bat "az webapp config set --resource-group $RESOURCE_GROUP --name $APP_SERVICE_NAME --startup-file=\\\"python -m uvicorn main:app --host 0.0.0.0 --port 8000\\\""

                    bat "az webapp deploy --resource-group $RESOURCE_GROUP --name $APP_SERVICE_NAME --src-path pythonapp.zip --type zip"
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Deployment Failed!'
        }
    }
}
