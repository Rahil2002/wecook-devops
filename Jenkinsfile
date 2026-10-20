pipeline {
    agent {
        label 'windows-docker'
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        DOCKERHUB_USER = 'rahil2002'
        BACKEND_IMAGE = 'rahil2002/wecook-backend'
        FRONTEND_IMAGE = 'rahil2002/wecook-frontend'
    }

    stages {
        stage('Checkout and Validate') {
            steps {
                bat 'git --version'
                bat 'docker version'
                bat 'kubectl config current-context'
                bat 'git status --short'
            }
        }

        stage('Build Images') {
            steps {
                bat 'docker build -t %BACKEND_IMAGE%:%BUILD_NUMBER% -t %BACKEND_IMAGE%:latest .\\backend'
                bat 'docker build -t %FRONTEND_IMAGE%:%BUILD_NUMBER% -t %FRONTEND_IMAGE%:latest .\\frontend'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    bat '@echo off\r\necho %DOCKER_PASSWORD%| docker login --username %DOCKER_USER% --password-stdin'
                }
            }
        }

        stage('Push Images') {
            steps {
                bat 'docker push %BACKEND_IMAGE%:%BUILD_NUMBER%'
                bat 'docker push %BACKEND_IMAGE%:latest'
                bat 'docker push %FRONTEND_IMAGE%:%BUILD_NUMBER%'
                bat 'docker push %FRONTEND_IMAGE%:latest'
            }
        }

        stage('Deploy to Minikube') {
            steps {
                bat 'kubectl set image deployment/wecook-backend backend=%BACKEND_IMAGE%:%BUILD_NUMBER%'
                bat 'kubectl set image deployment/wecook-frontend web=%FRONTEND_IMAGE%:%BUILD_NUMBER%'
                bat 'kubectl rollout status deployment/wecook-backend --timeout=180s'
                bat 'kubectl rollout status deployment/wecook-frontend --timeout=180s'
            }
        }

        stage('Verify Deployment') {
            steps {
                bat 'kubectl get deployments wecook-backend wecook-frontend'
                bat 'kubectl get pods'
            }
        }
    }

        stage('Verify Application Health') {
            steps {
                echo 'Checking WeCook frontend...'
                bat 'curl.exe --fail --silent --show-error --retry 10 --retry-delay 5 --retry-connrefused http://127.0.0.1/'

                echo 'Checking WeCook backend API...'
                bat 'curl.exe --fail --silent --show-error --retry 10 --retry-delay 5 --retry-connrefused http://127.0.0.1/api/recipes/public'

                echo 'Frontend and backend health checks passed.'
            }
        }

    post {
        always {
            bat 'docker logout'
        }
        success {
            echo 'WeCook CI/CD pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check the stage logs in Console Output.'
        }
    }
}
