pipeline {
    agent any

    stages {
        stage('Checkout Source') {
            steps {
                git credentialsId: 'github-pat', url: 'https://github.com/iadocicco-ale/pedelogo-catalogo.git', branch: 'main'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    dockerapp = docker.build("iadocicco/api-produto:${env.BUILD_ID}", 
                        '-f ./src/PedeLogo.Catalogo.Api/Dockerfile .')
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        dockerapp.push('latest')
                        dockerapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }

        stage('Deploy Kubernetes') {
            environment {
                tag_version = "${env.BUILD_ID}"
            }
            steps {
                script {
                    // Substitui {{tag}} pela versão gerada no build
                    sh 'sed -i "s/{{tag}}/${tag_version}/g" ./k8s/api/deployment.yaml'

                    // Exibe o YAML com a tag substituída
                    sh 'cat ./k8s/api/deployment.yaml'
                }

                // Aplica os manifestos no cluster
                withCredentials([file(credentialsId: 'kube', variable: 'KUBECONFIG')]) {
                    // API: deployment + service
                    sh 'kubectl apply -f ./k8s/api/'

                    // MongoDB: deployment + service
                    sh 'kubectl apply -f ./k8s/mongodb/'
                }
            }
        }
    }
}

