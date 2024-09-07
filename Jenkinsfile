pipeline {
    agent any
    
     tools {
        jdk 'jdk17'
        maven 'maven3'
    }
     environment{
        SCANNER_HOME=tool 'sonar'
    }
    

     stages {
        stage('Git Checkout') {
            steps {
                echo 'Git_checkout'
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/shubham9511s/Multi-Tier-With-Database.git'
            }
     }
     stage('Code quality check') {
            steps {
                echo 'Code_Quality_check'
                withSonarQubeEnv('sonar-server') {
                       sh"$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Bank-app -Dsonar.projectKey=Bank-app -Dsonar.java.binaries=. "
                      
                   }
            }
     }
     /*stage('Quality_Gate') {
               steps {
                 script{
                        def qualityGate = waitForQualityGate()
                        if (qualityGate.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qualityGate.status}"
                    }else{
                        
                        echo"Quality gate all condition are satisfy"
                    }
                }
          }
     }*/
     stage('Maven All Stages') {
            steps {
                echo 'Maven stage'
                sh 'mvn clean install -DskipTests'
            }
     }
     stage('Deploy to Artifactory') {
            environment {
                TARGET_REPO = 'Bank-jar'
            }
            
            steps {
                script {
                    try {
                        def server = Artifactory.newServer url: 'http://52.66.199.234:8082/artifactory', credentialsId: 'jfrog-token'
                        def uploadSpec = """{
                            "files": [
                                {
                                    "pattern": "target/*.jar",
                                    "target": "${TARGET_REPO}/"
                                }
                            ]
                        }"""
                        
                        server.upload(uploadSpec)
                    } catch (Exception e) {
                        error("Failed to deploy artifacts to Artifactory: ${e.message}")
                    }
                }
            }
        }
        
        stage('Build docker Image') {
            steps {
                    script{
                        withDockerRegistry(credentialsId: 'docker-token', toolName: 'docker') {
                            sh 'docker login -u shubhamshinde2206 -p dckr_pat_qz8FJ1XZlFWYfx79x4T3hLLRYdc '
                            sh'docker build -t shubhamshinde2206/bank-app:${BUILD_NUMBER} .'
                            
                        }
                    }
            }
        }
        stage('Trivy Image Scan ') {
            steps {
                    sh'trivy image --format table -o trivy-image-report.html shubhamshinde2206/bank-app:${BUILD_NUMBER}'
                
            }
        }
        
         stage('Push docker Image') {
            steps {
                    script{
                        withDockerRegistry(credentialsId: 'docker-token', toolName: 'docker') {
                            
                            sh'docker push shubhamshinde2206/bank-app:${BUILD_NUMBER}'
                            
                        }
                    }
            }
        }
        stage('Cleanup docker image') {
            steps {
                    script{
                        withDockerRegistry(credentialsId: 'docker-token', toolName: 'docker') {
                            
                            sh'docker rmi $(docker images -q)'
                            
                        }
                    }
            }
        }
     stage('Checkout Code from Manifest Repo-II') {
            steps {
                    git branch: 'main', changelog: false, poll: false, url: 'https://github.com/shubham9511s/manifest-file.git'
            }
        }
     stage('Update Deployment file') {
            environment {
                GIT_REPO_NAME = "manifest-file"
                GIT_USER_NAME = "shubham9511s"
                REPO_IMAGE_NAME = "shubhamshinde2206/bank-app"
            }
            steps {
                   withCredentials([string(credentialsId: 'github-token', variable: 'TOKEN')]) {
                        sh '''
                            git config user.email "shubham.ssc100@gmail.com"
                            git config user.name "shubham9511s"
                            BUILD_NUMBER=${BUILD_NUMBER}
                            echo $BUILD_NUMBER
                            imageTag=$(grep -oP '(?<=bank-app:)[^ ]+' bank-manifest/main.yml)
                            echo $imageTag
                            sed -i "s|${REPO_IMAGE_NAME}:${imageTag}|${REPO_IMAGE_NAME}:${BUILD_NUMBER}|" bank-manifest/main.yml
                            git add bank-manifest/main.yml
                            git clean -fd 
                            git commit -m "Update deployment Image to version \${BUILD_NUMBER}"
                            git push https://${TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                        '''
                    }
                }
            }
        
        
        
     }    
        
    }    
