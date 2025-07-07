pipeline {
    agent any

    environment{
        NPM_CONFIG_CACHE= "${WORKSPACE}/.npm"
        dockerImagePrefix = "us-west1-docker.pkg.dev/lab-agibiz/docker-repository"
        registry = "https://us-west1-docker.pkg.dev"
        registryCredentials = "gcp-registry"
    }

    stages {
        stage('Instalación de dependencias') {
            agent {
                docker {
                    image 'node:22'
                    reuseNode true
                }
            }
           stages {
                stage("Instalacion de Dependencias"){
                    steps {
                        sh 'npm ci'
                    }
                }
                stage("Inicio Testing"){
                    steps {
                        sh 'npm run test:cov'
                    }
                }
                stage("Deploy"){
                    steps {
                        sh 'npm run build'
                    }
                }
            }
        }
        stage ("Build y push Docker"){
            steps {
                script {
                    docker.withRegistry("${registry}", registryCredentials ){
                        sh "docker build -t backend-nest-test-sr ."
                        sh "docker tag backend-nest-sr ${dockerImagePrefix}/backend-nest-test-sr"
                        sh "docker push ${dockerImagePrefix}/backend-nest-test-sr"
                    }
                }
            }
        }
        stage ("Actualización de Kubernetes"){
            agent {
                docker {
                    image 'alpine/k8s:1.30.2'
                    reuseNode true
                }
            }
            steps {
                withKubeConfig([credentialsId: 'gcp-kubeconfig']){
                    sh 'kubectl -n lab-sr set image deployments/backend-nest-test-sr backend-nest-sr=us-west1-docker.pkg.dev/lab-agibiz/docker-repository/backend-nest-test-sr'
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline finalizado correctamente.'
        }
        failure {
            echo 'Error en el pipeline.'
        }
    }
}
