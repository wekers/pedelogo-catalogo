pipeline {
    agent any

    options {
        skipDefaultCheckout(true) // Desabilita checkout automático do SCM
    }

    environment {
        // Gera um timestamp único no formato YYYYMMDD-HHMMSS
        CUSTOM_TAG = sh(script: 'date +%Y%m%d-%H%M%S', returnStdout: true).trim()
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
                    dockerapp = docker.build("wekers/api-produto:${env.CUSTOM_TAG}",
                    '-f ./src/PedeLogo.Catalogo.Api/Dockerfile .')
                }
            }
        }
        stage('Push Image') {
            steps {
                script {
                        docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        dockerapp.push('latest')
                        dockerapp.push("${env.CUSTOM_TAG}")
                        }
                }
            }
        }
        stage('Deploy Kubernetes') {
            
            steps {
                script {
                    // Atualiza o YAML com a tag correta
                    "sed -i 's/{{tag}}/${env.CUSTOM_TAG}/g' ./k8s/api.yaml"

                    // Aplica a configuração usando kubectl
                    withCredentials([file(credentialsId: 'localkubeConfig', variable: 'KUBECONFIG')]) {
                        sh """
                            export KUBECONFIG=\${KUBECONFIG}
                            kubectl apply -f ./k8s/api.yaml
                            kubectl apply -f ./k8s/mongo.yaml
                        """
                    }
                }
            }
        }
    }
}
