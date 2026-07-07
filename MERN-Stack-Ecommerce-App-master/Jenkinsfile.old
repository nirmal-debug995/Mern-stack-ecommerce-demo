pipeline {
    agent any

    environment {
        ACR_NAME = "mernappacr"
        ACR_LOGIN_SERVER = "mernappacr.azurecr.io"

        FRONTEND_IMAGE = "fusion-frontend"
        BACKEND_IMAGE = "fusion-backend"
    }

    stages {

        stage('Build Frontend') {
            steps {
                dir('MERN-Stack-Ecommerce-App-master') {
                    sh '''
                    npm install
                    CI=false npm run build
                    '''
                }
            }
        }

        stage('Build Backend') {
            steps {
                dir('MERN-Stack-Ecommerce-App-master/backend') {
                    sh '''
                    npm install
                    '''
                }
            }
        }

        stage('Azure Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'azure-sp',
                    usernameVariable: 'AZURE_CLIENT_ID',
                    passwordVariable: 'AZURE_CLIENT_SECRET'
                )]) {

                    sh '''
                    az login --service-principal \
                      -u $AZURE_CLIENT_ID \
                      -p $AZURE_CLIENT_SECRET \
                      --tenant 99f0e748-181d-4fc2-88da-47a1569be0e2

                    az account set --subscription be01b6cb-fb41-465f-b65a-155a8942e8ba

                    az acr login --name ${ACR_NAME}
                    '''
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh """
                docker build -t ${ACR_LOGIN_SERVER}/${FRONTEND_IMAGE}:${BUILD_NUMBER} \
                -f MERN-Stack-Ecommerce-App-master/Dockerfile \
                MERN-Stack-Ecommerce-App-master

                docker build -t ${ACR_LOGIN_SERVER}/${BACKEND_IMAGE}:${BUILD_NUMBER} \
                -f MERN-Stack-Ecommerce-App-master/backend/Dockerfile \
                MERN-Stack-Ecommerce-App-master/backend
                """
            }
        }

        stage('Push Images') {
            steps {
                sh """
                docker push ${ACR_LOGIN_SERVER}/${FRONTEND_IMAGE}:${BUILD_NUMBER}
                docker push ${ACR_LOGIN_SERVER}/${BACKEND_IMAGE}:${BUILD_NUMBER}
                """
            }
        }

        stage('Deploy to AKS') {
            steps {
                sh """
                kubectl set image deployment/fusion-frontend \
                fusion-frontend=${ACR_LOGIN_SERVER}/${FRONTEND_IMAGE}:${BUILD_NUMBER}

                kubectl set image deployment/fusion-backend \
                fusion-backend=${ACR_LOGIN_SERVER}/${BACKEND_IMAGE}:${BUILD_NUMBER}

                kubectl rollout status deployment/fusion-frontend
                kubectl rollout status deployment/fusion-backend
                """
            }
        }
    }
}
