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
            agent {
                kubernetes {
                    cloud 'k8s-cluster01'
                }
            }
            environment {
                tag_version = "${env.BUILD_ID}"
            }
            steps {
                script {
                    // Substitui {{tag}} pela versão gerada no build
                    sh 'sed -i "s/{{tag}}/${tag_version}/g" ./k8s/api/deployment.yaml'

                    // Exibe o YAML com a versão injetada
                    sh 'cat ./k8s/api/deployment.yaml'

                    // Realiza o deploy no cluster via plugin Kubernetes CD
                    kubernetesDeploy(
                        configs: '**/k8s/**',
                        kubeconfigId: 'kube'
                    )
                }
            }
        }
    }
}
