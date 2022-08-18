// prerequisites: a nodejs app must be deployed inside a kubernetes cluster
//                repo must include the Dockerfile
//                docker pipeline must be installed on Jenkins
//                docker credentials must be created on Jenkins
// TODO: look for all instances of [] and replace all instances of 
//       '[VARIABLE]' with actual values 
//        e.g https://github.com/jeevansaichinta/events-app-api-server.git might become https://github.com/MyName/external.git
// variables:
// The following are common to both api-server and web-server
//      roidtc-aug22-u106
//      655ca1e8-1038-4d7e-a884-2c457d129cd3  //id of your global docker credentials in Jenkins https://www.google.com/search?q=add+docker+credentials+Jenkins&oq=add+docker+credentials+Jenkins
//      jeevansaichinta
//      events-app-cluster 
//      us-central1-c
// The following are codebase specific
//      https://github.com/jeevansaichinta/events-app-api-server.git
//	    main
//      api-server-image
//      the following values can be found in the yaml:
//      demo-api
//      demo-api (name of the container to be replaced - in the template/spec section of the deployment)


pipeline {
    agent any 
   environment {
        registryCredential = '655ca1e8-1038-4d7e-a884-2c457d129cd3'
        imageName = 'jeevansaichinta/api-server-image'
        dockerImage = ''
        }
    stages {
        stage('Get Source') {
             agent {
                docker { 
                    image 'node:14-alpine'
                    args '-e HOME=/tmp -e NPM_CONFIG_PREFIX=/tmp/.npm'
                    reuseNode true
                }
            }
            steps {
                echo 'Retrieving source from github' 
                git branch: 'main',
                    url: 'https://github.com/jeevansaichinta/events-app-api-server.git'
                echo 'Did we get the source?' 
                sh 'ls -a'
            }
        }
        stage('Install Dependencies') {
             agent {
                docker { 
                    image 'node:14-alpine'
                    args '-e HOME=/tmp -e NPM_CONFIG_PREFIX=/tmp/.npm'
                    reuseNode true
                }
            }
            steps {
                echo 'install dependencies' 
                sh 'npm install'
            }
        }
        stage('Run Tests') {
             agent {
                docker { 
                    image 'node:14-alpine'
                    args '-e HOME=/tmp -e NPM_CONFIG_PREFIX=/tmp/.npm'
                    reuseNode true
                }
            }
            steps {
                echo 'Run tests'
                sh 'npm test'
                echo 'Tests passed on to build and deploy Docker container'
            }
        }
        stage('Building image') {
            steps{
                script {
                    dockerImage = docker.build imageName
                }
            }
            }
        stage('Deploy Image') {
            steps{
                script {
                docker.withRegistry( '', registryCredential ) {
                    dockerImage.push("$BUILD_NUMBER")
                }
                }
            }
        }     
         stage('get k8s credentials') {
             agent {
                docker { 
                    image 'google/cloud-sdk:latest'
                    args "-e HOME=${WORKSPACE}/kube/"
                    reuseNode true
                    }
                    }
            steps {
                sh "rm -rf ${WORKSPACE}/kube/"
                echo 'Get cluster credentials'
                sh 'gcloud container clusters get-credentials events-app-cluster --zone us-central1-c --project roidtc-aug22-u106'
            }
        }     
         stage('update k8s') {
             agent {
                docker { 
                    image 'google/cloud-sdk:latest'
                    args '-e HOME=/tmp'
                    reuseNode true
                        }
                    }
            steps {
                echo 'Set the image'
                     sh "kubectl --kubeconfig=${WORKSPACE}/kube/.kube/config set image deployment/demo-api demo-api=${env.imageName}:${env.BUILD_NUMBER}"
            }
        }     
        stage('Remove local docker image') {
            steps{
                sh "docker rmi $imageName:latest"
                sh "docker rmi $imageName:$BUILD_NUMBER"
            }
        }
    }
}
