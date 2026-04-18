@Library('Shared')_
pipeline{
    agent {label 'node'}

        tools {
            maven 'Maven'
        }

    parameters{
        choice(name: 'DEPLOY_ENV', choices: ['blue','green'], description: 'Choose which environment to deploy: Blue or Green')
        choice(name: 'DOCKER_TAG', choices: ['blue','green'], description: 'Choose the Docker image tag for deployment')
        booleanParam(name: 'SWITCH_TRAFFIC', defaultValue: false, description: 'Switch traffic between Blue and Green')
    }

    environment {
        IMAGE_NAME = "kaustavsdet/bankapp"
        DOCKER_TAG = "${params.DOCKER_TAG}"
        KUBE_NAMESPACE = "webapps"
        SONAR_HOME = tool "Sonar"
    }

    stages{
        stage('Workspace cleanup'){
            steps{
                script{
                    clean_ws()
                }
            }
        }

        stage('Git checkout'){
            steps{
                script{
                    code_checkout('https://github.com/kaustavGitH/Blue-Green-Deployment.git','main')
                }
            }
        }

        stage('compile'){
            steps{
                sh "mvn compile"
            }
        }

        stage('test'){
            steps{
                sh "mvn test -DskipTests=true"
            }
        }

        stage('Trivy: Filescan system'){
            steps{
                script{
                    trivy_scan()
                }
            }
        }

        stage('Sonarqube: Code analysis'){
            steps{
                script{
                    sonarqube_analysis("Sonar","Blue Green Deploy","blue-green")
                }
            }
        }

        stage('Docker build'){
            steps{
                script{
                    docker_build(imageName: "${IMAGE_NAME}", imageTag: "${DOCKER_TAG}", dockerHubUser: "kaustavsdet")
                }
            }
        }

        stage('Docker push'){
            steps{
                script{
                    docker_push("${IMAGE_NAME}", "${DOCKER_TAG}")
                }
            }
        }

        stage('Deploy MySQL Deplyment and Service'){
            steps{
                script{
                    withKubeConfig(caCertificate: '', clusterName: 'blue-green-deploy.us-east-1.eksctl.io', contextName: '', credentialsId: 'k8s-cred', namespace: '${KUBE_NAMESPACE}', restrictKubeConfigAccess: false, serverUrl: 'https://024BD00864AD2FA18755C8A0D59A73ED.gr7.us-east-1.eks.amazonaws.com'){
                        sh "kubectl apply -f mysql-ds.yml -n ${KUBE_NAMESPACE}"
                    }
                }
            }
        }

        stage('Deploy SVC-APP'){
            steps{
                script{
                    withKubeConfig(caCertificate: '', clusterName: 'blue-green-deploy.us-east-1.eksctl.io', contextName: '', credentialsId: 'k8s-cred', namespace: '${KUBE_NAMESPACE}', restrictKubeConfigAccess: false, serverUrl: 'https://024BD00864AD2FA18755C8A0D59A73ED.gr7.us-east-1.eks.amazonaws.com'){
                        sh """
                            if ! kubectl get svc bankapp-service -n ${KUBE_NAMESPACE}; then
                                kubectl get svc bankapp-service.yml -n ${KUBE_NAMESPACE}
                            fi
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes'){
            steps{
                script{
                    def deploymentFile = ""
                    if(params.DEPLOY_ENV == blue){
                        deploymentFile = 'app-deployment-blue.yml'
                    }
                    else{
                        deploymentFile = 'app-deployment-green.yml'
                    }

                    withKubeConfig(caCertificate: '', clusterName: 'blue-green-deploy.us-east-1.eksctl.io', contextName: '', credentialsId: 'k8s-cred', namespace: '${KUBE_NAMESPACE}', restrictKubeConfigAccess: false, serverUrl: 'https://024BD00864AD2FA18755C8A0D59A73ED.gr7.us-east-1.eks.amazonaws.com'){
                        sh "kubectl apply -f ${deploymentFile} -n ${KUBE_NAMESPACE}"
                    }
                }
            }
        }

        stage('Switch traffic between Blue and Green'){
            when{
                expression{return params.SWITCH_TRAFFIC}
            }
            steps{
                script{
                    def newEnv = params.DEPLOY_ENV

                    withKubeConfig(caCertificate: '', clusterName: 'blue-green-deploy.us-east-1.eksctl.io', contextName: '', credentialsId: 'k8s-cred', namespace: '${KUBE_NAMESPACE}', restrictKubeConfigAccess: false, serverUrl: 'https://024BD00864AD2FA18755C8A0D59A73ED.gr7.us-east-1.eks.amazonaws.com'){
                        sh '''
                            kubectl patch service bankapp-service -p "{\\"spec\\":{\\"selector\\":{\\"app\\": \\"bankapp\\", \\"version\\": \\"'''+newEnv+'''\\"}}}" -n ${KUBE_NAMESPACE}
                        '''
                    }
                }
            }
        }

        stage('Verify deployment'){
            steps{
                script{
                    def verifyEnv = params.DEPLOY_ENV

                    withKubeConfig(caCertificate: '', clusterName: 'blue-green-deploy.us-east-1.eksctl.io', contextName: '', credentialsId: 'k8s-cred', namespace: 'gameapps', restrictKubeConfigAccess: false, serverUrl: 'https://024BD00864AD2FA18755C8A0D59A73ED.gr7.us-east-1.eks.amazonaws.com'){
                        sh """
                            kubectl get pods -l version=${verifyEnv} -n ${KUBE_NAMESPACE}
                            kubectl get svc bankapp-service -n ${KUBE_NAMESPACE}
                        """
                    }
                }
            }
        }
    }
}