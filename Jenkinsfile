pipeline {
  agent any
  tools { nodejs 'NodeJS' }   // must match the NodeJS tool name you configured in Jenkins

  environment {
    ARTIFACT_NAME       = 'app.zip'
    AZURE_WEBAPP_NAME   = 'luckywebapp'   // your App Service name
    AZURE_RESOURCE_GROUP= 'lucky'         // your Resource Group
  }

  stages {
    // NOTE: No explicit "Checkout from Git" stage.
    // Jenkins already did "Declarative: Checkout SCM" using your job's SCM + credentials.

    stage('Install Dependencies') {
      steps {
        // Prefer reproducible installs; fall back to npm install if no lockfile
        sh 'npm ci || npm install'
      }
    }

    stage('Package App') {
      steps {
        sh 'zip -r $ARTIFACT_NAME .'
      }
    }

    stage('Publish Artifact') {
      steps {
        archiveArtifacts artifacts: "${ARTIFACT_NAME}", fingerprint: true
      }
    }

    // Optional (you can delete this if build+deploy are in the same job)
    stage('Download Artifact') {
      steps {
        copyArtifacts projectName: env.JOB_NAME, filter: "${ARTIFACT_NAME}", fingerprintArtifacts: true
      }
    }

    stage('Login to Azure') {
      steps {
        withCredentials([
          usernamePassword(credentialsId: 'azure-sp', usernameVariable: 'AZURE_APP_ID', passwordVariable: 'AZURE_PASSWORD'),
          string(credentialsId: 'azure-tenant', variable: 'AZURE_TENANT')
        ]) {
          sh 'az login --service-principal -u $AZURE_APP_ID -p $AZURE_PASSWORD --tenant $AZURE_TENANT'
        }
      }
    }

    stage('Deploy to Azure Web App') {
      steps {
        sh 'echo "Deploying $ARTIFACT_NAME to $AZURE_WEBAPP_NAME" && az webapp deployment source config-zip --resource-group $AZURE_RESOURCE_GROUP --name $AZURE_WEBAPP_NAME --src $ARTIFACT_NAME'
      }
    }
  }
}
