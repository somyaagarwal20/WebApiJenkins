pipeline {
    agent any
    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal'
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
                bat 'dotnet restore'
                bat 'dotnet build --configuration Release'
                bat 'dotnet publish -c Release -o ./publish'
            }
        }

        stage('Azure Infrastructure Setup') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat "az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID"
                    
                    script {
                        // 1. Check/Create Resource Group
                        def rgOut = bat(script: "az group exists --name $RESOURCE_GROUP", returnStdout: true).trim()
                        if (rgOut == 'false') {
                            bat "az group create --name $RESOURCE_GROUP --location $LOCATION"
                            echo "Created Resource Group: $RESOURCE_GROUP"
                        }

                        // 2. Check/Create App Service Plan (Linux)
                        def planOut = bat(script: "az appservice plan list --resource-group $RESOURCE_GROUP --query \"[?name=='$APP_SERVICE_PLAN'].name\" -o tsv", returnStdout: true).trim()
                        if (planOut != "$APP_SERVICE_PLAN") {
                            bat "az appservice plan create --name $APP_SERVICE_PLAN --resource-group $RESOURCE_GROUP --sku F1 --is-linux --location $LOCATION"
                            echo "Created App Service Plan: $APP_SERVICE_PLAN"
                        }

                        // 3. Check/Create Web App (.NET 8)
                        def webAppOut = bat(script: "az webapp list --resource-group $RESOURCE_GROUP --query \"[?name=='$APP_SERVICE_NAME'].name\" -o tsv", returnStdout: true).trim()
                        if (webAppOut != "$APP_SERVICE_NAME") {
                            bat "az webapp create --name $APP_SERVICE_NAME --resource-group $RESOURCE_GROUP --plan $APP_SERVICE_PLAN --runtime \"DOTNETCORE:8.0\""
                            echo "Created Web App: $APP_SERVICE_NAME"
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                bat "az webapp deploy --resource-group $RESOURCE_GROUP --name $APP_SERVICE_NAME --src-path ./publish --type zip"
            }
        }
    }
}
