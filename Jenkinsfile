pipeline {

    agent any
    //escenarios -> escenario -> pasos
    environment{
        NPM_CONFIG_CACHE="${WORKSPACE}/.npm"
        dockerimagePrefix = "us-west1-docker.pkg.dev/lab-agibiz/docker-repository"
        registry = "https://us-west1-docker.pkg.dev"
        registryCredentials = 'gcp-registry'
    }
    stages{

        stage ("proceso de build y test") {
            agent {
                docker {
                    image 'node:22'
                    reuseNode true
                }
            }

            stages {
                stage("instalacion de dependencias"){
                    steps{
                        sh 'npm ci'
                    }
                }
                stage("test"){
                    steps{
                        sh 'npm run test:cov'
                    }
                }
                stage("build de la aplicacion"){
                    steps{
                        sh 'npm run build'
                    }
                }

            }

        }
        stage ("build y push de imagen docker"){
            steps{
                script{
                    docker.withRegistry("${registry}", registryCredentials){
                        sh "docker build -t backend-nest-test-sr ."
                        sh "docker tag backend-nest-test-sr ${dockerimagePrefix}/backend-nest-test-sr"
                        sh "docker tag backend-nest-test-sr ${dockerimagePrefix}/backend-nest-test-sr:${BUILD_NUMBER}"
                        sh "docker push ${dockerimagePrefix}/backend-nest-test-sr"
                        sh "docker push ${dockerimagePrefix}/backend-nest-test-sr:${BUILD_NUMBER}"
                    }
                }

            }
        }
        stage ("actualizacion de kubernetes"){
            agent {
                docker {
                    image 'alpine/k8s:1.30.2'
                    reuseNode true
                }
            }
            steps {
                withKubeConfig([credentialsId: 'gcp-kubeconfig']){
                    sh 'kubectl -n lab-test-sr set image deployments/backend-nest-test-sr backend-nest-test-sr=${dockerimagePrefix}/backend-nest-test-sr:${BUILD_NUMBER}'
                }
            }
        }
    }
}