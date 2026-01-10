pipeline {
  agent any

  environment {
    AWS_REGION = "ap-south-1"
    ECR_REPO = "331760067638.dkr.ecr.ap-south-1.amazonaws.com/prod-nginx"
    IMAGE_TAG = "v${BUILD_NUMBER}"
  }

  stages {
    stage('Clean Workspace') {
      steps {
         deleteDir()
      }
    }

    stage('Checkout') {
      steps {
        git url: 'https://github.com/bk-thakur/nginx-ci-repo.git', branch: 'main'
      }
    }

    stage('Build Image') {
      steps {
        sh 'docker build -t prod-nginx:${IMAGE_TAG} .'
      }
    }

    stage('Push Image to ECR') {
      steps {
        sh '''
        aws ecr get-login-password --region $AWS_REGION |
        docker login --username AWS --password-stdin $ECR_REPO
        docker tag prod-nginx:${IMAGE_TAG} $ECR_REPO:${IMAGE_TAG}
        docker push $ECR_REPO:${IMAGE_TAG}
        '''
      }
    }

    stage('Update GitOps Repo') {
      steps {
        sh '''
        git clone https://github.com/bk-thakur/nginx-cd-repo.git
        cd nginx-cd-repo/prod
        sed -i "s|IMAGE_TAG|$ECR_REPO:$IMAGE_TAG|" deployment.yaml
        git commit -am "Release $IMAGE_TAG"
        git push origin main
        '''
      }
    }
  }
}
