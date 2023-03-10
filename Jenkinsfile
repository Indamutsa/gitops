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
                git branch: 'dev', 
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
        // Placing this in another pipeline to avoid having config files in the root of the workspace
        // ------------------------------------------------------------------------------------------
        // stage('Updating Kubernetes deployment file'){
        //     steps {
        //         sh "cat deployment.yml"
        //         sh "sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yml"
        //         sh "cat deployment.yml"
        //     }
        // }
        // stage('Push the changed deployment file to Git'){
        //     steps {
        //         script{
        //             sh """
        //             git config --global user.name "indamutsa"
        //             git config --global user.email "arsichizy@gmail.com"
        //             git add deployment.yml
        //             git commit -m 'Updated the deployment file' """

        //             withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'pass', usernameVariable: 'user')]) {
        //                 sh "git push https://$user:$pass@github.com/Indamutsa/gitops.git dev"
        //             }
        //         }
        //     }
        // }
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





// Now the final pipeline
// ---------------------------

/**

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
                git branch: 'dev', 
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
        
        stage('Trigger config change pipeline'){
            steps {
                sh "curl -v -k arsene:1170503463a0d88ab3a2578a616ca8628e -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'https://057f-77-228-3-139.eu.ngrok.io/job/gitops-config/buildWithParameters?token=gitops-config'"
            }
        }
    }
}






*/