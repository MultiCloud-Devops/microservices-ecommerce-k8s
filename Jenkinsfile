pipeline {
    agent any 

    environment  {
        AWS_ACCOUNT_ID = credentials('ACCOUNT_ID')
        AWS_ECR_REPO_NAME = "adservice"
        AWS_DEFAULT_REGION = 'ap-south-1'
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/"
    }

    stages {
        stage('Cleaning Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/MultiCloud-Devops/microservices-ecommerce-k8s.git'
            }
        }
        stage("Docker Image Build") {
            steps {
                script {
                    dir('src/adservice') {
                        sh 'docker system prune -f'
                        sh 'docker container prune -f'
                        sh 'docker build -t ${AWS_ECR_REPO_NAME} .'
                    }
                }
            }
        }
        stage("ECR Image Pushing") {
            steps {
                script {
                        sh 'aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${REPOSITORY_URI}'
                        sh 'docker tag ${AWS_ECR_REPO_NAME} ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}'
                        sh 'docker push ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}'
                }
            }
        }
        stage('Update Deployment file') {
            environment {
                GIT_REPO_NAME = "microservices-ecommerce-k8s"
                GIT_USER_NAME = "MultiCloud-Devops"
            }
            steps {
                    withCredentials([string(credentialsId: 'git_token', variable: 'git_token')]) {
                        sh '''
                            git config user.email "krishnaurs2022@gmail.com"
                            git config user.name "MultiCloud-Devops"
                            BUILD_NUMBER=${BUILD_NUMBER}
                            echo $BUILD_NUMBER
                            sed -i "/name: $AWS_ECR_REPO_NAME/,/image:/s#image:.*#image: ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}#g" kuberenetes-manfest-file.yaml
                            git add .
                            git commit -m "Update deployment Image to version \${BUILD_NUMBER}"
                            git push https://${git_token}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                        '''
                    }
                }
            }
        }
    }

