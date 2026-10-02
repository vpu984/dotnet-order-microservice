pipeline {

    agent any

    environment {
        DOCKER_IMAGE = 'vpu984/order-service'
        DOCKER_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                bat '''
                docker run --rm ^
                  -v "%WORKSPACE%:/src" ^
                  -w /src ^
                  mcr.microsoft.com/dotnet/sdk:8.0 ^
                  sh -c "dotnet restore && dotnet build --configuration Release && dotnet test --configuration Release --no-build"
                '''
            }
        }

        stage('Docker Build') {
            steps {
                bat "docker build -t %DOCKER_IMAGE%:%DOCKER_TAG% ."
                bat "docker tag %DOCKER_IMAGE%:%DOCKER_TAG% %DOCKER_IMAGE%:latest"
            }
        }

        stage('Docker Login and Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    bat '''
                        echo %DOCKER_PASSWORD% | docker login -u %DOCKER_USERNAME% --password-stdin
                        docker push %DOCKER_IMAGE%:%DOCKER_TAG%
                        docker push %DOCKER_IMAGE%:latest
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'OrderService CI/CD pipeline completed successfully.'
        }

        failure {
            echo 'OrderService pipeline failed. Check the console output.'
        }
    }
}
