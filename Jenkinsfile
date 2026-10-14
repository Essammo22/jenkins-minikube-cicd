pipeline {
    agent any

    environment {
        IMAGE = "essammo22/myapp"
        TAG   = "${BUILD_NUMBER}"
    }

    stages {
        stage('Build Image') {
            steps {
                sh 'docker build -t $IMAGE:$TAG -t $IMAGE:latest .'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_PASS')]) {
                    sh '''
                      echo $DH_PASS | docker login -u $DH_USER --password-stdin
                      docker push $IMAGE:$TAG
                      docker push $IMAGE:latest
                    '''
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                withCredentials([file(credentialsId: 'minikube-kubeconfig',
                                      variable: 'KUBECONFIG')]) {
                    sh '''
                      sed "s|myapp:latest|myapp:$TAG|; s|DOCKERHUB_USER/myapp|$IMAGE|" K8s/deployment.yaml | kubectl apply -f -
                      kubectl apply -f K8s/service.yaml
                      kubectl rollout status deployment/myapp
                    '''
                }
            }
        }
    }
}
