pipeline {
   agent any

   environment {
     // You must set the following environment variables
     // ORGANIZATION_NAME
     // YOUR_DOCKERHUB_USERNAME (it doesn't matter if you don't have one) - a ja mam i super!
     registryCredential = 'dockerhub'
     dockerImage = ''
     SERVICE_NAME = "fleetman-api-gateway"
     REPOSITORY_TAG = "${YOUR_DOCKERHUB_USERNAME}/${ORGANIZATION_NAME}-${SERVICE_NAME}:${BUILD_ID}"
   }

   stages {
      stage('Preparation') {
         steps {
            cleanWs()
            git credentialsId: 'GitHub', url: "https://github.com/${ORGANIZATION_NAME}/${SERVICE_NAME}"
         }
      }
      stage('Build') {
         steps {
            sh '''mvn clean package'''
         }
      }
 
    stage('Build Image') {
         steps {
           // sh 'docker image build -t ${REPOSITORY_TAG} .'
            script { 
                    dockerImage = docker.build REPOSITORY_TAG 
                }
         }
      }
      
      stage('Push Image') {
         steps {
            script { 
               docker.withRegistry( '', registryCredential ) { 
                        dockerImage.push() 
                    }
                }
         }
      }

      stage('Deploy to Cluster') {
        //   steps {       
        //      sh 'envsubst < ${WORKSPACE}/deploy.yaml | kubectl apply -f -'
        //  } 
             steps {
                  script {
                    withAWS(credentials: 'AWS-CREDS', region: 'eu-central-1') {
                       sh 'aws eks update-kubeconfig --name fleetman --region eu-central-1' 
                       sh 'envsubst < ${WORKSPACE}/deploy.yaml | kubectl apply -f -'
                        }   
                  }
             }           
   }
}
}   
