pipeline {
    agent any

    environment {
        AWS_REGION     = 'us-east-2'
        AWS_ACCOUNT_ID = '458697684235'
        ECR_REPOSITORY = 'tech-challenge-2-app'
        ECR_REGISTRY   = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        EKS_CLUSTER    = 'tech-challenge-2-eks'
        HELM_RELEASE   = 'hello-world'
        HELM_CHART     = './helm/hello-world'
        K8S_NAMESPACE  = 'hello-world'
    }

    stages {
        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                      -t $ECR_REPOSITORY:$BUILD_NUMBER \
                      ./app
                '''
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sh '''
                    ECR_PASSWORD=$(aws ecr get-login-password --region $AWS_REGION)

                    docker login \
                      --username AWS \
                      --password "$ECR_PASSWORD" \
                      $ECR_REGISTRY
                '''
            }
        }

        stage('Tag and Push Image') {
            steps {
                sh '''
                    docker tag \
                      $ECR_REPOSITORY:$BUILD_NUMBER \
                      $ECR_REGISTRY/$ECR_REPOSITORY:$BUILD_NUMBER

                    docker push \
                      $ECR_REGISTRY/$ECR_REPOSITORY:$BUILD_NUMBER
                '''
            }
        }

        stage('Configure kubectl') {
            steps {
                sh '''
                    aws eks update-kubeconfig \
                      --region $AWS_REGION \
                      --name $EKS_CLUSTER
                '''
            }
        }

        stage('Deploy with Helm') {
            steps {
                sh '''
                    helm upgrade --install $HELM_RELEASE $HELM_CHART \
                      --namespace $K8S_NAMESPACE \
                      --create-namespace \
                      --set image.repository=$ECR_REGISTRY/$ECR_REPOSITORY \
                      --set image.tag=$BUILD_NUMBER
                '''
            }
        }

        stage('Wait for Rollout') {
            steps {
                sh '''
                    kubectl rollout status \
                      deployment/$HELM_RELEASE \
                      -n $K8S_NAMESPACE \
                      --timeout=300s
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl get pods -n $K8S_NAMESPACE
                    kubectl get service -n $K8S_NAMESPACE
                    kubectl get ingress -n $K8S_NAMESPACE
                    kubectl get hpa -n $K8S_NAMESPACE
                '''
            }
        }
    }

    post {
        success {
            echo 'Jenkins CI/CD deployment completed successfully.'
        }

        failure {
            echo 'Jenkins CI/CD deployment failed. Review the console output.'
        }
    }
}