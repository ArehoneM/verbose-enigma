pipeline {
    agent any

    environment {
        REGISTRY = "localhost:5000"
        APP_NAME = "my-app"
        VERSION = "v1.0"
    }

    stages {
        stage('Build Image') {
            steps {
                script {
                    docker.build("${REGISTRY}/${APP_NAME}:${VERSION}")
                }
            }
        }

        stage('Push Image & Tag for Environments') {
            steps {
                script {
                    docker.withRegistry("http://${REGISTRY}", "") {
                        docker.image("${REGISTRY}/${APP_NAME}:${VERSION}").push()

                        // Taggin for env
                        def envs = ['uat', 'stg', 'prod']
                        for (env in envs) {
                            docker.image("${REGISTRY}/${APP_NAME}:${VERSION}").tag("${REGISTRY}/${APP_NAME}:${VERSION}-${env}")
                            docker.image("${REGISTRY}/${APP_NAME}:${VERSION}-${env}").push()
                        }
                    }
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    sh "kubectl apply -f k8s/deployment-uat.yaml"
                    sh "kubectl apply -f k8s/deployment-stg.yaml"
                    sh "kubectl apply -f k8s/deployment-prod.yaml"
                }
            }
        }
    }
}
