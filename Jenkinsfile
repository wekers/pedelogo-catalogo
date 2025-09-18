pipeline {
    agent any

    options {
        skipDefaultCheckout(true) // Desabilita checkout automático do SCM
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
        stage('Deploy Kubernetes') {
            environment {
                TAG_VERSION = "${env.BUILD_ID}"
            }
            steps {
                script {
                    // Atualiza o YAML com a tag correta
                    sh "sed -i 's/{{tag}}/${TAG_VERSION}/g' ./k8s/api.yaml"

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
