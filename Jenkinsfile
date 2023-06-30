pipeline{

    agent any
    
    stages{

        stage('Build and Test') {
            steps {
                sh 'ls -ltr'
                // build the project and create a JAR file
                sh 'mvn clean package'

                
            }

            post {
                success {
                    script {
                        slackSend(
                        color: '#36a64f',  
                        message: "Build ${env.BUILD_NUMBER} succeeded!",  
                        CredentialId: 'slack-jenkins'  
                        )
                    }
                }
            }
                
        }

        stage("Static Code Analysis"){

            environment {
                SONAR_URL = "https://0b73-2405-201-a405-1893-d587-7c74-d639-3cdb.ngrok-free.app"
                }

            steps{
                echo "========Start Testing========"
                
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                    // sh 'sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
                    sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
                    
                }

            }

            post {
                success {
                    script {
                        slackSend(
                            color: '#36a64f',  
                            message: "Test of ${env.BUILD_NUMBER} succeeded! using sonarqube",  
                            tokenCredentialId: 'slack-jenkins'  
                        )
                    }
                }

                failure {
                    script {
                        slackSend(
                            color: '#ff0000',
                            message: "Test failed",
                            tokenCredentialId: 'slack-jenkins'
                        )

                    }
                }
            }
        }

        stage("Build docker file and push"){

            environment {
                DOCKER_IMAGE = "sushank3/ci_cd-portfolio:v${BUILD_NUMBER}"
                // DOCKERFILE_LOCATION = "java-maven-sonar-argocd-helm-k8s/spring-boot-app/Dockerfile"
                REGISTRY_CREDENTIALS = credentials('docker_hub')
            
            }

            steps{

                script {
                    sh "docker build -t ${DOCKER_IMAGE} ."
                    def dockerImage = docker.image("${DOCKER_IMAGE}")
                    docker.withRegistry("https://index.docker.io/v1/", "docker_hub") {
                        dockerImage.push()
                    }

                    slackSend(
                        color: '#36a64f',  
                        message: "Docker build ${env.BUILD_NUMBER} succeeded!",  
                        tokenCredentialId: 'slack-jenkins'  
                    )
                }
            }
        }

        stage('Update Deployment File') {

            steps {
                    sh 'echo passed'
                    git branch: 'main', url: 'https://gitlab.com/sushank3/ci_cd-portfolio.git'
                }

            // stage('Checkout') {
            //     steps {
            //         sh 'echo passed'
            //         git branch: 'main', url: 'https://gitlab.com/sushank3/ci_cd-portfolio.git'
            //     }
            // }

            environment {
                GITLAB_PROJECT_NAME = "CI_CD-portfolio"
                GITLAB_USERNAME = "sushank3"
            }
        
            steps {
                withCredentials([string(credentialsId: 'gitlab-token', variable: 'GITLAB_TOKEN')]) {
                    sh '''
                        git config user.email "sushankkr3@gmail.com"
                        git config user.name "sushank3"
                        
                        BUILD_NUMBER="${BUILD_NUMBER}"

                        CURRENT_IMAGE_VERSION=$(grep -oE 'sushank3/ci_cd-portfolio:v[0-9]+' deployment.yaml | cut -d':' -f2)
                        sed -i "s/${CURRENT_IMAGE_VERSION}/v${BUILD_NUMBER}/g" deployment.yaml
                        git add deployment.yaml
                        
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                        
                        git push "https://oauth2:${GITLAB_TOKEN}@gitlab.com/${GITLAB_USERNAME}/${GITLAB_PROJECT_NAME}" HEAD:main


                    '''

                }

                // withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                //     sh '''
                //         git config user.email "sushankkr3@gmail.com"
                //         git config user.name "Sushank Kumar"
                        
                //         BUILD_NUMBER="${BUILD_NUMBER}"

                //         // CURRENT_IMAGE_VERSION=$(grep -oE 'sushank3/ci_cd-portfolio:v[0-9]+' deployment.yaml | cut -d':' -f2)

                        

                //         sed -i "s/${CURRENT_IMAGE_VERSION}/v${BUILD_NUMBER}/g" deployment.yaml
                        
                    
                //         git add deployment.yaml
                        
                //         git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                        
                //         git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    
                //     '''
                // }

                script {
                    slackSend(
                        color: '#36a64f',  
                        message: "Final Build ${env.BUILD_NUMBER} succeeded! and deployment file updated with new version",  
                        tokenCredentialId: 'slack-jenkins'  
                    )
                }
            }
        }
    }
    
}
