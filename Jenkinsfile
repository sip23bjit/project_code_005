pipeline {
    agent any
    tools {
        maven 'MAVEN'
    }
    environment {
        DOCKERHUB_CREDS = ''
        DOCKER_IMAGE = 'sip23bjit/sp04'
        REGISTRY = 'registry.hub.docker.com'
    }
    triggers {
        pollSCM 'H/5 * * * *'
    }
    stages {
        stage('Cloning Git') {
            steps {
                git branch: 'main', url: 'https://github.com/sip23bjit/project_code_005.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Building Docker image') {
            steps {
                script {
                    lms=docker.build(DOCKER_IMAGE)
                }
            }
        }

        stage('Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub') {
                        lms.push("${env.BUILD_ID}")
                        lms.push("latest")
                    }
                }
            }
        }

        stage('Remove Unused docker image') {
            steps {
                script {
                    sh "docker rmi ${DOCKER_IMAGE}:latest"
                    sh "docker rmi ${DOCKER_IMAGE}:${env.BUILD_ID}"
                }
            }
        }

        stage("Integration of k8s") {
            steps {
                script {
                    withKubeConfig(credentialsId: 'kubeconfig', context: 'default') {
                        sh 'kubectl get pods'
                        sh 'kubectl delete -f 01.yaml 2>/dev/null || true'
                        sh 'sleep 100'
                        sh 'kubectl apply -f 01.yaml'
                    }
                }
            }
        }
    }
}
