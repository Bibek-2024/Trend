pipeline {
agent any

```
environment {
    IMAGE_NAME = "bibekdec2022/trend-app"
    AWS_REGION = "ap-south-1"
    EKS_CLUSTER = "trend-cluster"
}

stages {

    stage('Checkout Code') {
        steps {
            checkout scm
        }
    }

    stage('Install Dependencies') {
        steps {
            sh '''
            if [ -f package.json ]; then
                npm install
            fi
            '''
        }
    }

    stage('Build React App') {
        steps {
            sh '''
            if [ -f package.json ]; then
                npm run build
            fi
            '''
        }
    }

    stage('Build Docker Image') {
        steps {
            sh '''
            docker build -t $IMAGE_NAME:latest .
            '''
        }
    }

    stage('DockerHub Login') {
        steps {
            withCredentials([usernamePassword(
                credentialsId: 'dockerhub-creds',
                usernameVariable: 'DOCKER_USER',
                passwordVariable: 'DOCKER_PASS'
            )]) {
                sh '''
                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                '''
            }
        }
    }

    stage('Push Docker Image') {
        steps {
            sh '''
            docker push $IMAGE_NAME:latest
            '''
        }
    }

    stage('Deploy to Kubernetes') {
        steps {
            withCredentials([[
                $class: 'AmazonWebServicesCredentialsBinding',
                credentialsId: 'aws-eks'
            ]]) {
                sh '''
                aws eks --region $AWS_REGION update-kubeconfig --name $EKS_CLUSTER

                kubectl apply -f deployment.yaml
                kubectl apply -f service.yaml

                kubectl rollout status deployment/trend-app-deployment
                '''
            }
        }
    }

    stage('Verify Deployment') {
        steps {
            sh '''
            kubectl get pods
            kubectl get svc
            '''
        }
    }

}

post {
    always {
        sh 'docker system prune -f'
    }
}
```

}
