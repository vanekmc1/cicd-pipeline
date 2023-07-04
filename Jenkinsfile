pipeline {
    agent any
    tools {nodejs "node"}
    parameters {
        choice(name: "BRANCH", choices: ["dev", "main"], description: "Branch")
        string(name: "IMAGE_TAG", defaultValue: "v1.0", trim: false, description: "Image Tag")
    }
    stages {
        stage('Assign Variables') {
            steps {
                script {
                    def BRANCH = env.BRANCH_NAME ?: params.BRANCH  
                    def IMAGE_TAG = params.IMAGE_TAG
                    
                    env.BRANCH = BRANCH
                    env.IMAGE_TAG = IMAGE_TAG

                    echo "Branch: ${BRANCH}"
                    echo "Image tag: ${IMAGE_TAG}"
                }
            }
        }
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH}", url: 'https://github.com/vanekmc1/cicd-pipeline.git'
            }
        }     
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                sh 'CI=true npm test'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh "docker build -t node${env.BRANCH}:${env.IMAGE_TAG} ."
            }
        }
        stage('Deploy') {
            steps {
                script {
                    def PORT
                    switch(env.BRANCH) { 
                        case "main": PORT = '3000'; break
                        case "dev": PORT = '3001'; break
                    }            
                    sh "docker rm -f node${env.BRANCH} || echo 'No container found'"
                    sh "docker run -d --name node${env.BRANCH} --expose ${PORT} -p ${PORT}:3000 node${env.BRANCH}:${env.IMAGE_TAG}"
                }
            }
        }
    }
}
