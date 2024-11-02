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
                    def serviceUrl = ""
                    def maxAttempts = 5
                    // Intentos para obtener la URL del LoadBalancer
                    for (int i = 0; i < maxAttempts; i++) {
                        serviceUrl = sh(script: "kubectl get svc nginx-game-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'", returnStdout: true).trim()
                        if (serviceUrl) {
                            echo "Service URL: http://${serviceUrl}"
                            break
                        } else {
                            echo "Attempt ${i + 1} of ${maxAttempts}: Waiting for the LoadBalancer IP..."
                            sleep 60
                        }
                    }
                    // Verificar si se obtuvo la URL o si se alcanzaron los intentos máximos
                    if (!serviceUrl) {
                        error "Failed to get the LoadBalancer IP after ${maxAttempts} attempts."
                    }
                    
                }
            }
        }
       
      
    }
}