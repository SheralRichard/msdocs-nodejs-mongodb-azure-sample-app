pipeline {
  agent any
  tools { nodejs 'NodeJS' }   // must match NodeJS tool name in Jenkins

  environment {
    ARTIFACT_NAME = 'app.zip'
    AZURE_WEBAPP_NAME = 'luckywebapp'         // your App Service name
    AZURE_RESOURCE_GROUP = 'lucky'            // your Resource Group name
  }

  stages {
    stage('Checkout from Git') {
      steps {
        git branch: 'main', url: 'https://github.com/SheralRichard/msdocs-nodejs-mongodb.git'
      }
    }

    stage('Install Dependencies') {
      steps { sh 'npm install' }
    }

    stage('Package App') {
      steps { sh 'zip -r $ARTIFACT_NAME .' }
    }

    stage('Publish Artifact') {
      steps { archiveArtifacts artifacts: "${ARTIFACT_NAME}", fingerprint: true }
    }

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
