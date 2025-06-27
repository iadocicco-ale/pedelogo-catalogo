pipeline {
    agent any

    stages {
        stage('Checkout Source') {
            steps {
                git url: 'https://github.com/fabricioveronez/pedelogo-catalogo.git', branch: 'main'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    dockerapp = docker.build("fabricioveronez/api-produto:${env.BUILD_ID}", 
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
    }
}
