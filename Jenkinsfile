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
                script {
                    bat 'dotnet restore'
                    bat 'dotnet build --configuration Release'
                    bat 'dotnet publish -c Release -o publish'
                }
            }
        }

        stage('Azure Setup') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat "az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID"
                }
                
                script {
                    def rgExists = bat(script: "az group exists --name $RESOURCE_GROUP", returnStdout: true).trim() == 'true'
                    if (!rgExists) { 
                        bat "az group create --name $RESOURCE_GROUP --location $LOCATION" 
                    }
                    
                    def planExists = bat(script: "az appservice plan show --name $APP_SERVICE_PLAN --resource-group $RESOURCE_GROUP --query name -o tsv", returnStdout: true).trim() == "$APP_SERVICE_PLAN"
                    if (!planExists) { 
                        bat "az appservice plan create --name $APP_SERVICE_PLAN --resource-group $RESOURCE_GROUP --sku F1 --is-linux" 
                    }
                    
                    def webAppExists = bat(script: "az webapp show --name $APP_SERVICE_NAME --resource-group $RESOURCE_GROUP --query name -o tsv", returnStdout: true).trim() == "$APP_SERVICE_NAME"
                    if (!webAppExists) { 
                        bat "az webapp create --name $APP_SERVICE_NAME --resource-group $RESOURCE_GROUP --plan $APP_SERVICE_PLAN --runtime 'DOTNETCORE:8.0'" 
                    }
                }
            }
        }
        
        stage('Deploy to Azure') {
            steps {
                bat "az webapp deploy --resource-group $RESOURCE_GROUP --name $APP_SERVICE_NAME --src-path ./publish.zip --type zip"
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
