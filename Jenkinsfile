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

        stage('Azure Infrastructure Check') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat "az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID"
                    
                    script {
                        // 1. Verify Resource Group (working)
                        def rgExists = bat(script: "az group exists --name $RESOURCE_GROUP", returnStdout: true).trim() == 'true'
                        if (!rgExists) {
                            error "Resource Group $RESOURCE_GROUP not found - please create it first"
                        }
        
                        // 2. Bulletproof App Service Plan Check
                        def planCheck = bat(
                            script: "az appservice plan show --name $APP_SERVICE_PLAN --resource-group $RESOURCE_GROUP --query \"name\" -o tsv",
                            returnStdout: true
                        ).trim()
                        
                        if (planCheck != "$APP_SERVICE_PLAN") {
                            error "App Service Plan $APP_SERVICE_PLAN not found in Resource Group $RESOURCE_GROUP"
                        }
        
                        // 3. Web App Check
                        def webAppCheck = bat(
                            script: "az webapp show --name $APP_SERVICE_NAME --resource-group $RESOURCE_GROUP --query \"name\" -o tsv",
                            returnStdout: true
                        ).trim()
                        
                        if (webAppCheck != "$APP_SERVICE_NAME") {
                            bat "az webapp create --name $APP_SERVICE_NAME --resource-group $RESOURCE_GROUP --plan $APP_SERVICE_PLAN --runtime 'DOTNETCORE:8.0'"
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
