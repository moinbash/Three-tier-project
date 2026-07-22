pipeline {
    agent any
    environment {
        BACKEND_IMAGE  = "moin9592/mern-backend"
        FRONTEND_IMAGE = "moin9592/mern-frontend"
        TAG = "${BUILD_NUMBER}"
        // Jenkins credentials IDs
        DOCKER_CREDS = "dockerhub-creds"
        // GitHub Repo
        GIT_URL = "https://github.com/moinbash/Three-tier-project.git"
    }
    stages {
        stage('Checkout Code from GitHub') {
            steps {
                git branch: "main", url: "${GIT_URL}"
            }
        }
        stage('Build Docker Images') {
            steps {
                dir('mern/backend') {
                    bat "docker build -t %BACKEND_IMAGE%:%TAG% ."
                }
                dir('mern/frontend') {
                    bat "docker build -t %FRONTEND_IMAGE%:%TAG% ."
                }
            }
        }
        stage('Push Images to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: DOCKER_CREDS,
                    usernameVariable: "DOCKER_USER",
                    passwordVariable: "DOCKER_PASS"
                )]) {
                    bat '''
                    docker login -u %DOCKER_USER% -p %DOCKER_PASS%
                    docker push %BACKEND_IMAGE%:%TAG%
                    docker push %FRONTEND_IMAGE%:%TAG%
                    docker logout
                    '''
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                bat '''
                echo ===== Checking kubeconfig =====
                set KUBECONFIG=C:\\Users\\ASUS\\.kube\\config
                minikube update-context
                kubectl config current-context
                kubectl get nodes
                powershell -Command "(Get-Content k8s.yaml) -replace '%BACKEND_IMAGE%:.*', '%BACKEND_IMAGE%:%TAG%' | Set-Content k8s.yaml"
                powershell -Command "(Get-Content k8s.yaml) -replace '%FRONTEND_IMAGE%:.*', '%FRONTEND_IMAGE%:%TAG%' | Set-Content k8s.yaml"
                kubectl apply -f k8s.yaml
                kubectl rollout restart deployment mongodb-deployment
                kubectl rollout restart deployment backend-deployment
                kubectl rollout restart deployment frontend-deployment
                '''
            }
        }
    }
}
