pipeline {
    agent any
    environment {
        DOCKERHUB_USERNAME = "indamutsa"
        APP_NAME = "gitops-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}" + "/" + "${APP_NAME}"
        REGISTRY_CREDS = 'dockerhub'
        }
    stages {
        stage('Cleanup Workspace'){
            steps {
                script {
                    cleanWs()
                }
            }
        }
        stage('Checkout SCM'){
            steps {
                git branch: 'main', 
                credentialsId: 'github', 
                url: 'https://github.com/Indamutsa/gitops.git'
            }
        }
        stage('Build Docker Image'){
            steps {
                script{
                    docker_image = docker.build "${IMAGE_NAME}"
                }
            }
        }
        stage('Push Docker Image'){
            steps {
                script{
                    docker.withRegistry('', REGISTRY_CREDS ){
                        docker_image.push("${BUILD_NUMBER}")
                        docker_image.push('latest')
                    }
                }
            }
        } 
        stage('Delete Docker Images'){
            steps {
                sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                sh "docker rmi ${IMAGE_NAME}:latest"
            }
        }
        stage('Updating Kubernetes deployment file'){
            steps {
                sh "cat deployment.yml"
                sh "sed -i 's/${APP_NAME}.*/${IMAGE_NAME}:${IMAGE_TAG}/g' deployment.yml"
                sh "cat deployment.yml"
            }
        }
        stage('Push the changed deployment file to Git'){
            steps {
                script{
                    sh """
                    git config --global user.name "indamutsa"
                    git config --global user.email "arsichizy@gmail.com"
                    git add deployment.yml
                    git commit -m 'Updated the deployment file' """

                    withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'pass', usernameVariable: 'user')]) {
                        sh "echo $user $pass"
                    }
                }
            }
        }
    }
}


// stage('Build Docker Image'){
//             steps {
//                 sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
//                 sh "docker build -t ${IMAGE_NAME}:latest ."
//             }
//         }
//         stage('Push Docker Image'){
//             steps {
//                 withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'pass', usernameVariable: 'user')]) {
//                     sh "docker login -u $user --password $pass"
//                     sh "docker push ${IMAGE_NAME}:${IMAGE_TAG} ."
//                     sh "docker push ${IMAGE_NAME}:latest ."
//                 }
//             }
//         }