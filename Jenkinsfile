pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'bxr', url: 'https://github.com/werwerTrain/eurekaServer.git'
            }
        }
        
        stage('delete old image in k8s'){
            steps{
                 bat '''
                kubectl delete -f k8s/eurekaServer-deployment.yaml || true
                kubectl delete -f k8s/eurekaServer-service.yaml || true
                '''
            }
        }
        stage('Build new image') {
            steps {
                script {
                    powershell '''
                    $containers = docker ps -q --filter "ancestor=bxr0820/eurekaserver:latest"
                    foreach ($container in $containers) {
                        Write-Output "Stopping container $container"
                        docker stop $container
                    }
    
                    $allContainers = docker ps -a -q --filter "ancestor=bxr0820/eurekaserver:latest"
                    foreach ($container in $allContainers) {
                        Write-Output "Removing container $container"
                        docker rm $container
                    }
                    '''
                    bat 'docker rmi -f bxr0820/eurekaserver:latest || true'
                    bat '''
                    docker build -t bxr0820/eurekaserver .
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                        bat '''
                        echo buxinran123| docker login -u bxr0820 --password-stdin
                        docker push bxr0820/eurekaserver:latest
                        '''
                }
            }
        }


        stage('deploy to k8s'){
            steps{
                bat '''
                kubectl apply -f k8s/eurekaServer-deployment.yaml
                kubectl apply -f k8s/eurekaServer-service.yaml
                kubectl apply -f k8s/eurekaServer-hpa.yaml
                '''
                echo '部署成功'
            }
        }
    }

    post {
        always {
            bat 'docker system prune -f'
        }
        success {
            echo 'Build and deployment succeeded!'
        }
        failure {
            echo 'Build or deployment failed.'
        }
    }
}
