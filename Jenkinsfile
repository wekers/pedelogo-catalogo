pipeline {
    agent any

    options {
        // garante que o workspace seja limpo antes de cada execução
        cleanWs()
    }

    stages {
        stage('Checkout Source') {
            steps {
                // Usa as configurações de SCM já definidas no job
                checkout scm
                //git url:'https://github.com/wekers/pedelogo-catalogo.git', branch:'main'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    dockerapp = docker.build("wekers/api-produto:${env.BUILD_ID}",
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
