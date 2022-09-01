pipeline {
    agent any 
    environment {
        registryCredential = 'dockerhub'
        imageName = 'stanleys/externalapp'
        dockerImage = ''
	PATH = "/usr/sbin:/usr/bin:/sbin:/bin:${env.PATH}"
        }
    stages {
        stage('Run the tests') {
             agent {
                docker { 
                    image 'node:14-alpine'
                    args '-e HOME=/tmp -e NPM_CONFIG_PREFIX=/tmp/.npm'
		    args "-u root"  
                    alwaysPull true
                    reuseNode true
                }
            }
            steps {
                echo 'Retrieve source from github. run npm install and npm test' 
                git branch: 'main',
                    url: 'https://github.com/movinglightspeed/devops_ui_svc.git'
                echo 'repo files'
                sh 'ls -a'
            }
        }
        stage('Building image') {
            steps{
                script {
                    echo 'build the image'
		    dockerImage = docker.build("${env.imageName}:${env.BUILD_ID}")
	            echo "${env.imageName}:${env.BUILD_ID}"
                    echo 'image built'
                }
            }
            }
        stage('Push Image') {
            steps{
                script {
                    echo 'push the image to docker hub'
                    docker.withRegistry('',registryCredential){
                        dockerImage.push("${env.BUILD_ID}")
                  }
                }
            }
        }     
         stage('deploy to k8s') {
             agent {
                docker { 
                    image 'ubuntu:latest'
                    args '-u root'          
                    reuseNode true
                    alwaysPull true
                        }
                    }
            steps {
                echo 'Get cluster credentials'
		sh 'apt-get update -y && apt-get upgrade -y && apt-get install curl -y && apt-get install -y gnupg && apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 871920D1991BC93C'
                sh 'apt --fix-broken install -y'
                sh 'apt-get install apt-transport-https ca-certificates gnupg -y'
                sh 'echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list'
                sh 'curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key --keyring /usr/share/keyrings/cloud.google.gpg add -'
                sh 'apt-get update -y && apt-get install google-cloud-cli -y'      
                sh 'apt-get install google-cloud-sdk-gke-gcloud-auth-plugin -y'
                sh 'apt-get install -y kubectl'
                sh 'gcloud auth activate-service-account ${gaccount} --key-file=devopsbootcamp-355721-d2c37704b9c8.json'
                sh 'gcloud config set account ${gaccount}'
		sh 'gcloud container clusters get-credentials cluster-1 --zone us-central1-c --project devopsbootcamp-355721'
		sh "kubectl create deployment devops-ui-svc --image=${env.imageName}:${env.BUILD_ID}"
              }
            }       
        stage('Remove local docker images') {
            steps{
                script {
                    echo 'push the image to docker hub' 
                }
                // sh "docker rmi $imageName:latest"
                sh "docker rmi $imageName:$BUILD_NUMBER"
            }
        }
    }
}
