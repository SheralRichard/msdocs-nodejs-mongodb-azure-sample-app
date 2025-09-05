pipeline {
  agent any

  tools { nodejs 'NodeJS' }  // must match the NodeJS tool name you configured in Jenkins

  environment {
    ARTIFACT_NAME        = 'app.zip'
    AZURE_WEBAPP_NAME    = 'luckywebapp'   // your App Service name
    AZURE_RESOURCE_GROUP = 'lucky-rg'      // updated resource group name
  }

  stages {
    // Jenkins automatically does "Declarative: Checkout SCM"

    stage('Install Dependencies') {
      steps {
        sh 'npm ci || npm install'
      }
    }

    stage('Package App') {
      steps {
        sh 'zip -r "$ARTIFACT_NAME" .'
      }
    }

    stage('Publish Artifact') {
      steps {
        archiveArtifacts artifacts: "${ARTIFACT_NAME}", fingerprint: true
      }
    }

    stage('Login to Azure') {
      steps {
        withCredentials([
          usernamePassword(credentialsId: 'azure-sp', usernameVariable: 'AZURE_APP_ID', passwordVariable: 'AZURE_PASSWORD'),
          string(credentialsId: 'azure-tenant', variable: 'AZURE_TENANT')
        ]) {
          sh 'az login --service-principal -u "$AZURE_APP_ID" -p "$AZURE_PASSWORD" --tenant "$AZURE_TENANT"'
        }
      }
    }

    stage('Deploy to Azure Web App') {
      steps {
        sh 'echo "Deploying $ARTIFACT_NAME to $AZURE_WEBAPP_NAME in $AZURE_RESOURCE_GROUP" && az webapp deploy --resource-group "$AZURE_RESOURCE_GROUP" --name "$AZURE_WEBAPP_NAME" --src-path "$ARTIFACT_NAME" --type zip'
      }
    }
  }

  post {
    always {
      echo 'Pipeline finished.'
    }
  }
}
