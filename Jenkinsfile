pipeline {
    agent any

    environment {
        IMAGE_TAG   = ""
        FULL_IMAGE  = ""
        REGISTRY    = "192.168.56.10:5000"
        APP_NAME    = "apps-example-v1"
        INFRA_REPO_URL = "https://github.com/yogaarie/template.git"
        CREDENTIALS_ID = "yogaarie"
    }

    stages {
        stage('Setup & Checkout') {
            steps {
                script {
                    // 1. Clone Application Repo (into the root workspace)
                    git branch: 'main', 
                        credentialsId: "${CREDENTIALS_ID}", 
                        url: "https://github.com/yogaarie/${APP_NAME}.git"
                    
                    // 2. Clone Infrastructure/Template Repo (into a specific folder)
                    dir('infra-templates') {
                        git branch: 'master', 
                            credentialsId: "${CREDENTIALS_ID}", 
                            url: "${INFRA_REPO_URL}"
                    }

                    // 3. Generate the tag from the App Repo (current directory)
                    IMAGE_TAG = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    FULL_IMAGE = "${REGISTRY}/test/${APP_NAME}:${IMAGE_TAG}"
                    
                    echo "Target Image: ${FULL_IMAGE}"
                }
            }
        }

        stage('Build & Push Docker') {
            steps {
                // Docker build runs in the root workspace where the App Dockerfile is
                sh "docker build -t ${FULL_IMAGE} ."
                sh "docker push ${FULL_IMAGE}"
            }
        }

        stage('Update & Push Manifest') {
            steps {
                // Move back into the infra folder cloned in stage 1
                dir('infra-templates') {
                    script {
                        def valuesPath = "chart-helm/values.yaml"
                        
                        // Update the tag
                        sh "sed -i 's/^\\s*tag: .*/  tag: \"${IMAGE_TAG}\"/' ${valuesPath}"
                        
                        // Git config and commit
                        sh """
                            git config user.email "yogaarir@gmail.com"
                            git config user.name "yoga"
                            git add ${valuesPath}
                            git commit -m "chore: update image tag to ${IMAGE_TAG} [skip ci]" || echo "No changes"
                        """
                        
                        // Secure Push using token from credentials
                        withCredentials([usernamePassword(
                            credentialsId: "${CREDENTIALS_ID}", 
                            passwordVariable: 'GIT_PASSWORD', 
                            usernameVariable: 'GIT_USERNAME'
                        )]) {
                            sh 'git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/yogaarie/template.git master'
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs(notFailBuild: true)
        }
    }
}