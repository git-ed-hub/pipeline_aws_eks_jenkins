pipeline {
    agent any
    environment {
	    APP_NAME = "nginx-game"
        RELEASE = "1.0.0"
        DOCKER_USER = "testsysadmin8"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        EKS_CLUSTER = "my-eks"
        AWS_REGION = "us-east-1"

    }
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/git-ed-hub/pipeline_aws_eks_jenkins.git'
                }
        }

        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }

       }

       stage ('Cleanup Artifacts') {
           steps {
               script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
               }
          }
       }
       stage('K8S Deploy') {
            steps {
                script {
                    withAWS(credentials: 'AWS-CREDS', region: "${AWS_REGION}") {
                        sh "aws eks update-kubeconfig --name ${EKS_CLUSTER} --region ${AWS_REGION}"
                        sh 'kubectl apply -f EKS-deployment.yaml'
                    }
                }
            }
        }

        stage('Get Service URL') {
            steps {
                script {
                    withAWS(credentials: 'AWS-CREDS', region: "${AWS_REGION}") {
                        def serviceUrl = ""
                        // Wait for the LoadBalancer IP to be assigned
                        timeout(time: 5, unit: 'MINUTES') {
                            while(serviceUrl == "") {
                                serviceUrl = sh(script: "kubectl get svc word-counter-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'", returnStdout: true).trim()
                                if(serviceUrl == "") {
                                    echo "Waiting for the LoadBalancer IP..."
                                    sleep 10
                                }
                            }
                        }
                        echo "Service URL: http://${serviceUrl}"
                    }
                }
            }
        }
       
      
    }
}