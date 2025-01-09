pipeline {
    agent any

    environment {
        IMAGE_NAME = "rafallltm/pedelogo-catalogo"
    }

    stages {
        stage('Checkout Código') {
            steps {
                echo 'Fazendo checkout da branch rtm do repositório público...'
                git branch: 'rtm', url: 'https://github.com/rafallltm/pedelogo-catalogo.git'
            }
        }

        stage('Build image') {
            steps {
                script {
                    dockerapp = docker.build("${IMAGE_NAME}:${env.BUILD_ID}", "-f ./src/PedeLogo.Catalogo.Api/Dockerfile .")
                }
            }
        }

        stage('Push image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        dockerapp.push('latest')
                        dockerapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo 'Deploying MongoDB...'
                    withCredentials([file(credentialsId: 'k8s-local', variable: 'KUBECONFIG')]) {
                        sh 'kubectl apply -f k8s/mongodb-deployment.yaml --kubeconfig=$KUBECONFIG'
                    }
                    echo 'Deploying API...'
                    withCredentials([file(credentialsId: 'k8s-local', variable: 'KUBECONFIG')]) {
                        sh 'kubectl apply -f k8s/app-deployment.yaml --kubeconfig=$KUBECONFIG'
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finalizado.'
        }
        success {
            echo 'Pipeline concluído com sucesso!'
        }
        failure {
            echo 'Pipeline falhou. Verifique os logs.'
        }
    }
}