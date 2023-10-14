pipeline {
    agent any

    stages {
        stage ('Build') {
            steps {
                script {
                    dockerapp = docker.build("paulofponciano/webgo:${env.BUILD_ID}", '-f ./Dockerfile ./') 
                }                
            }
        }

        stage ('Push') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        dockerapp.push('latest')
                        dockerapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }

        stage ('Deploy') {
            environment {
                tag_version = "${env.BUILD_ID}"
            }
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/kustomization.yaml'
                    sh 'kubectl apply -k ./k8s'
                }
            }
        }
    }
}