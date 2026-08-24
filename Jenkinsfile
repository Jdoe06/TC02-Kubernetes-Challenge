pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '822424645857'
        ECR_REPOSITORY = 'tc02-hello-world'
        EKS_CLUSTER = 'tc02-eks-cluster'
        IMAGE_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                      -t $ECR_REPOSITORY:$BUILD_NUMBER \
                      ./app
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login \
                      --username AWS \
                      --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh '''
                    docker tag \
                      $ECR_REPOSITORY:$BUILD_NUMBER \
                      $IMAGE_URI:$BUILD_NUMBER
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                    docker push $IMAGE_URI:$BUILD_NUMBER
                '''
            }
        }

        stage('Connect to EKS') {
            steps {
                sh '''
                    aws eks update-kubeconfig \
                      --region $AWS_REGION \
                      --name $EKS_CLUSTER

                    kubectl get nodes
                '''
            }
        }

        stage('Deploy with Helm') {
            steps {
                sh '''
                    helm upgrade --install tc02-hello-world ./k8s/tc02-hello-world \
                      --set image.repository=$IMAGE_URI \
                      --set image.tag=$BUILD_NUMBER
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl rollout status deployment/tc02-hello-world
                    kubectl get pods
                    kubectl get svc
                    kubectl get ingress
                    kubectl get hpa
                '''
            }
        }
    }
}