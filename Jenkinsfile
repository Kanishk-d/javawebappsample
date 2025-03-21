pipeline {
  agent any

  environment {
    AZURE_SUBSCRIPTION_ID = 'a778bbf9-56d1-4e99-b19a-0ecd2b322cf8'
    AZURE_TENANT_ID       = 'a9e61550-cf79-4349-83d3-dec5440cc70c'
    RESOURCE_GROUP        = 'jenkins-get-started-rg'
    WEBAPP_NAME           = 'jenkins-demo-app123'
    WAR_FILE              = 'target/calculator-1.0.war'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        sh 'mvn clean package'
      }
    }

    stage('Deploy to Azure') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'AzureServicePrincipal',
          usernameVariable: 'AZURE_CLIENT_ID',
          passwordVariable: 'AZURE_CLIENT_SECRET'
        )]) {
          sh '''
            az login --service-principal \
              --username "$AZURE_CLIENT_ID" \
              --password "$AZURE_CLIENT_SECRET" \
              --tenant "$AZURE_TENANT_ID"

            az account set --subscription "$AZURE_SUBSCRIPTION_ID"

            az webapp deploy \
              --resource-group "$RESOURCE_GROUP" \
              --name "$WEBAPP_NAME" \
              --src-path "$WAR_FILE" \
              --type war

            az logout
          '''
        }
      }
    }
  }

  post {
    success {
      echo '✅ Deployment completed successfully!'
    }
    failure {
      echo '❌ Deployment failed. Please check logs.'
    }
  }
}
