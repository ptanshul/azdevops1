pipeline {
  agent any
  
  environment {
    AZURE_SUBSCRIPTION_ID = 'your-subscription-id'
    AZURE_TENANT_ID = 'your-tenant-id'
    AZURE_CREDENTIAL_ID = 'your-azure-credential-id'
    ACR_NAME = 'myACRName'
    AKS_RESOURCE_GROUP = 'myResourceGroup'
    AKS_CLUSTER_NAME = 'myAKSCluster'
    
    DOCKER_IMAGE_NAME = "${ACR_NAME}.azurecr.io/myapp"
    DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"
  }
  
  stages {
    stage('Build Docker Image') {
      steps {
        script {
          docker.build("${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}")
        }
      }
    }
    
    stage('Push to ACR') {
      steps {
        withCredentials([azureServicePrincipal(AZURE_CREDENTIAL_ID)]) {
          sh '''
            az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
            az acr login --name $ACR_NAME
            docker push ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
          '''
        }
      }
    }
    
    stage('Deploy to AKS') {
      steps {
        withCredentials([azureServicePrincipal(AZURE_CREDENTIAL_ID)]) {
          sh '''
            az aks get-credentials --resource-group $AKS_RESOURCE_GROUP --name $AKS_CLUSTER_NAME
            kubectl set image deployment/myapp myapp=${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
          '''
        }
      }
    }
  }
}