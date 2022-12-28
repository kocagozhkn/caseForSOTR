pipeline {
    agent any
    tools {
        maven 'Maven3'
      
    }
    environment {
        DATE = new Date().format('yy.M')
        TAG = "${DATE}.${BUILD_NUMBER}"
    }
    stages {
        
        stage("Git Checkout"){
            steps {
                git branch: 'main',url:"https://github.com/kocagozhkn/case-vdfn"
            }
        }
        
        stage ('Build') {
            steps {
                sh 'mvn clean package'
                
            }
        }
        
        stage('Docker Build') {
            steps {
                script {
                    docker.build("kocagoz/case-vdfn:${TAG}")
                }
            }
        }
	    stage('Pushing Docker Image to Dockerhub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_credential') {
                       
                        docker.image("kocagoz/case-vdfn:${TAG}").push("latest")
                    }
                }
            }
        }
       stage('Deployment') {
            steps {
            sh 'kubectl --insecure-skip-tls-verify apply -f ./k8s-deployment'
            sh 'kubectl rollout restart deployment'
            }
        }
    }
}