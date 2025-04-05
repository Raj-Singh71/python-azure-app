pipeline {
    agent any

    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal-p'
        RESOURCE_GROUP = 'rg-jenkins'
        APP_SERVICE_NAME = 'jenkinspythonfastapi12387685'
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
                bat ".\\venv\\Scripts\\pip install fastapi uvicorn[standard] gunicorn"
                bat ".\\venv\\Scripts\\pip freeze > requirements.txt"
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

                    // Required for Linux App Service
                    bat "az webapp config appsettings set --resource-group $RESOURCE_GROUP --name $APP_SERVICE_NAME --settings WEBSITES_PORT=8000"

                    // Correct Linux-compatible startup command
                    bat "az webapp config set --resource-group $RESOURCE_GROUP --name $APP_SERVICE_NAME --startup-file \"gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app --bind 0.0.0.0:8000\""

                    // Deploy the zip
                    bat "az webapp deploy --resource-group $RESOURCE_GROUP --name $APP_SERVICE_NAME --src-path pythonapp.zip --type zip"

                    // Optional but recommended
                    bat "az webapp restart --name $APP_SERVICE_NAME --resource-group $RESOURCE_GROUP"
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
