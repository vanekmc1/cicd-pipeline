pipeline {
    agent any
    tools {nodejs "node"}
    parameters {
        choice(name: "BRANCH", choices: ["dev", "main"], description: "Branch")
        string(name: "IMAGE_TAG", defaultValue: "v1.0", trim: false, description: "Image Tag")
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('DOCKER_HUB')
    }
    stages {
        stage('Assign Variables') {
            steps {
                script {
                    def BRANCH = env.BRANCH_NAME ?: params.BRANCH  
                    def IMAGE_TAG = params.IMAGE_TAG
                    
                    env.BRANCH = BRANCH
                    env.IMAGE_TAG = IMAGE_TAG

                    library("utils@${env.BRANCH}")
                    log.info "Branch: ${BRANCH}"
                    log.info "Image tag: ${IMAGE_TAG}"
                }
            }
        }    
        stage('Build') {
            agent {
                docker {
                    image 'node'
                }
            }
            steps {
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                sh 'CI=true npm test'
            }
        }
        stage ("Hadolint") {
            agent {
                docker {
                    image 'hadolint/hadolint:latest-debian'
                }
            }
            steps {
                sh 'hadolint Dockerfile | tee -a hadolint_lint.txt'
            }
            post {
                always {
                    archiveArtifacts 'hadolint_lint.txt'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                sh "docker build -t vanekmc1/node${env.BRANCH}:${env.IMAGE_TAG} ."
            }
        }
        stage('Push Docker Image') {
            steps {
                sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
                sh "docker push vanekmc1/node${env.BRANCH}:${env.IMAGE_TAG}"
                sh "docker logout"
            }
        }
        stage('Trivy') {
            steps {
                script {
                    def vulnerabilities = sh(script: "docker run --rm bitnami/trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress vanekmc1/node${env.BRANCH}:${env.IMAGE_TAG}", returnStdout: true).trim()
                    echo "Vulnerability Report:\n${vulnerabilities}"
                }
            }
        }

        stage ('Deploy'){
            steps {
                build job: "Deploy_to_${env.BRANCH}", wait:true, parameters: [string(name: 'IMAGE_TAG' , value: "${params.IMAGE_TAG}" )]
            }
        }
    }
}
