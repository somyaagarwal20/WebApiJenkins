pipeline {
    agent any

    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal'  // Set this in Jenkins Credentials
        RESOURCE_GROUP = 'rg-jenkins'
        APP_SERVICE_PLAN = 'asp-jenkins'
        APP_SERVICE_NAME = 'webapijenkins8372648'
        LOCATION = 'EastUS2'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/Cloud-Authority/WebApiJenkins.git'
            }
        }

        stage('Build') {
            steps {
                script {
                    bat 'dotnet restore'
                    bat 'dotnet build --configuration Release'
                    bat 'dotnet publish -c Release -o publish'
                }
            }
        }

        stage('Create Azure Resources') {
            steps {
                script {
                    withCredentials([azureServicePrincipal(AZURE_CREDENTIALS_ID)]) {
                        bat """
                        az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID

                        REM Check if Resource Group exists
                        if not exist az group show --name $RESOURCE_GROUP >nul 2>&1 (
                            az group create --name $RESOURCE_GROUP --location $LOCATION
                        ) else (
                            echo Resource Group $RESOURCE_GROUP already exists
                        )

                        REM Check if App Service Plan exists
                        if not exist az appservice plan show --name $APP_SERVICE_PLAN --resource-group $RESOURCE_GROUP >nul 2>&1 (
                            az appservice plan create --name $APP_SERVICE_PLAN --resource-group $RESOURCE_GROUP --sku B1 --is-linux
                        ) else (
                            echo App Service Plan $APP_SERVICE_PLAN already exists
                        )

                        REM Check if Web App exists
                        if not exist az webapp show --name $APP_SERVICE_NAME --resource-group $RESOURCE_GROUP >nul 2>&1 (
                            az webapp create --name $APP_SERVICE_NAME --plan $APP_SERVICE_PLAN --resource-group $RESOURCE_GROUP --runtime "DOTNET:8.0"
                        ) else (
                            echo Web App $APP_SERVICE_NAME already exists
                        )
                        """
                    }
                }
            }
        }

        stage('Deploy to Azure App Service') {
            steps {
                script {
                    withCredentials([azureServicePrincipal(AZURE_CREDENTIALS_ID)]) {
                        bat """
                        az webapp deploy --resource-group $RESOURCE_GROUP --name $APP_SERVICE_NAME --src-path ./publish.zip --type zip
                        """
                    }
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
