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
            agent {
                Kubernetes {
                    cloud 'localKubernetes'
                }
            }
            enviroment {
                tag_version = "${env.BUILD_ID}"
            }
            steps {
                sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/api.yaml'
                sh 'cat ./k8s/api.yaml'
                kubernetesDeploy(configs: '**/k8s/**', kubeconfigId: 'kubeconfig')
            }
        }
    }
}
