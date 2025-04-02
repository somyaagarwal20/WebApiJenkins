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

        stage('Azure Setup') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat "az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID"
                }
                
                script {
                    def rgExists = bat(script: "az group show --name $AZURE_RESOURCE_GROUP", returnStatus: true) == 0
                    if (!rgExists) {
                        bat "az group create --name $AZURE_RESOURCE_GROUP --location $AZURE_LOCATION"
                    }
                    
                    def planExists = bat(script: "az appservice plan show --name $AZURE_PLAN_NAME --resource-group $AZURE_RESOURCE_GROUP", returnStatus: true) == 0
                    if (!planExists) {
                        bat "az appservice plan create --name $AZURE_PLAN_NAME --resource-group $AZURE_RESOURCE_GROUP --sku F1"
                    }
                    
                    def webAppExists = bat(script: "az webapp show --name $AZURE_APP_NAME --resource-group $AZURE_RESOURCE_GROUP", returnStatus: true) == 0
                    if (!webAppExists) {
                        bat "az webapp create --name $AZURE_APP_NAME --resource-group $AZURE_RESOURCE_GROUP --plan $AZURE_PLAN_NAME --runtime 'DOTNET:8.0'"
                    }
                }
            }
        }
        
        stage('Deploy to Azure') {
            steps {
                bat "az webapp deploy --resource-group $AZURE_RESOURCE_GROUP --name $AZURE_APP_NAME --src-path publish_output"
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
