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
                        az group create --name $RESOURCE_GROUP --location $LOCATION
                        az appservice plan create --name $APP_SERVICE_PLAN --resource-group $RESOURCE_GROUP --sku B1 --is-linux
                        az webapp create --name $APP_SERVICE_NAME --plan $APP_SERVICE_PLAN --resource-group $RESOURCE_GROUP --runtime "DOTNET:8.0"
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
