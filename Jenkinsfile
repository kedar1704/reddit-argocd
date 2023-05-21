pipeline {
    agent any

    environment {
        DOCKER_USERNAME = "kedar1704"
        APP_NAME = "gitops-argo-reddit-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKER_USERNAME}" + "/" + "${APP_NAME}" + ":" + "${IMAGE_TAG}"
        REGISTRY_CREDS = "dockerhub"
    }

    stages{
        stage("Cleaning the Workspace"){
            steps {
                script {
                    cleanWs()
                }
            }
        }
        stage("Git Checkout of Code"){
            steps{
                script{
                    checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/kedar1704/gitops_argocd_project.git']]])
                }
            }
        }
        stage("Build Docker image"){
            steps{
                script{
                    docker_image = docker.build "${IMAGE_NAME}"
                }
            }
        }
        stage("Push image to dockerhub registry"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'dockerhub') {
                        docker_image.push()
                        docker_image.push('latest')
                    }
                }
            }
        }
        stage("Delete Docker Images"){
            steps{
                sh "docker rmi -f ${IMAGE_NAME}"
            }
        }
        stage("Updating Kubernetes deployment"){
            steps{
                script{
                    sh """
                        cat deployment.yaml
                        sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yaml
                        cat deployment.yaml
                        
                       """
                }
            }
        }
        stage("Push Changed Deployment file to Github"){
            steps{
                script{
                    sh """
                       git config --global user.name "kedar1704"
                       git config --global user.email "bhaleraokedar.17@gmail.com"
                       git add deployment.yaml
                       git commit -m "New deployment file"

                       """
                    withCredentials([gitUsernamePassword(credentialsId: 'github-argocd', gitToolName: 'Default')]) {
                        sh """
                            git branch 
                            git push -u origin HEAD:main

                           """
                    }
                }
            }
        }
    }
}
