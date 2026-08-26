pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '822424645857'
        ECR_REPOSITORY = 'tc02-hello-world'
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

        stage('Update GitOps Image Tag') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'github-token',
                        usernameVariable: 'GITHUB_USER',
                        passwordVariable: 'GITHUB_TOKEN'
                    )
                ]) {
                    sh '''
                        sed -i "s/tag: .*/tag: \\"$BUILD_NUMBER\\"/" \
                          k8s/tc02-hello-world/values.yaml

                        git config user.name "Jenkins"
                        git config user.email "jenkins@tc02.local"

                        git add k8s/tc02-hello-world/values.yaml

                        git commit \
                          -m "Update application image to build $BUILD_NUMBER"

                        git push \
                          https://$GITHUB_USER:$GITHUB_TOKEN@github.com/Jdoe06/TC02-Kubernetes-Challenge.git \
                          HEAD:main
                    '''
                }
            }
        }
    }
}