pipeline {
    agent {
        label 'Docker-Node'
    }

    environment {
        KUBECONFIG_CREDENTIAL_ID = 'k8s-kubeconfig-dev'
        version = "backend_${env.BUILD_NUMBER}"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/persevcareers/NodeJS-Backend.git'
            }
        }

       stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "echo \"$DOCKER_PASSWORD\" | sudo docker login --username \"$DOCKER_USERNAME\" --password-stdin"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def dockerfilePath = '.'
                   sh "sudo docker build -t 'persevcareers6577/perseverance-project:${version}' ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh "sudo docker push 'persevcareers6577/perseverance-project:${version}'"
                }
            }
        } 

        stage('Deploy to Kubernetes') {
            steps {

                withCredentials([file(credentialsId: KUBECONFIG_CREDENTIAL_ID, variable: 'KUBECONFIG')]) {
                    script {
                        def kubeconfigPath = env.KUBECONFIG
                        def version = 'backend_${env.BUILD_NUMBER}'
                      withEnv(["VERSION=${env.version}"])
                      {
                         sh "echo ${VERSION}"
                         sh "export KUBECONFIG=${kubeconfigPath}"
                         sh "kubectl scale deploy api --replicas=0 -n three-tier"
                         sh" sed -i 's/VERSION/${VERSION}/g' backend.yml"
                         sh " cat backend.yml"
                         sh "kubectl apply -f backend.yml --validate=false"
                         sh "kubectl get pods -n three-tier"
                         sh "kubectl get svc -n three-tier"
                      }
                    }
                }
            }
        }
    }
}