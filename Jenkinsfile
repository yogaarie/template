pipeline {
    agent any

    environment {
        // Registry credentials ID in Jenkins (create one if you haven't)
        REGISTRY_CREDS = 'nexus' 
        REGISTRY       = "192.168.56.10:5000"
        APP_NAME       = "apps-example-v1"
        INFRA_REPO_URL     = "https://github.com/yogaarie/template.git"
        GIT_CREDS      = "yogaarie"
    }

    stages {
stage('Checkout') {
    steps {
        script {
            // Manual Clone for App
            git branch: 'main', 
                credentialsId: "${GIT_CREDS}", 
                url: "https://github.com/yogaarie/${APP_NAME}.git"

            // Manual Clone for Infra
            dir('infra-templates') {
                git branch: 'master', 
                    credentialsId: "${GIT_CREDS}", 
                    url: "${INFRA_REPO_URL}"
            }
        }
    }
}

        stage('Build & Push') {
            steps {
                script {
                    docker.withRegistry("http://${REGISTRY}", "${REGISTRY_CREDS}") {
                        // Builds and tags based on the branch parameter
                        def customImage = docker.build("${APP_NAME}:${params.branch}")
                        customImage.push()
                    }
                }
            }
        }

        stage('Update Manifests') {
            steps {
                dir('infra-templates') {
                    script {
                        def valuesPath = "chart-helm/values.yaml"
                        
                        // Use yq to update values safely
                        sh """
                           yq -i -y '.image.repository = "${REGISTRY}/${APP_NAME}"' ${valuesPath}
                           yq -i -y '.image.tag = "${params.branch}"' ${valuesPath}
                        """
                        
                        // Git commit and push
                        withCredentials([usernamePassword(credentialsId: "${GIT_CREDS}", passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                            sh """
                                git config user.email "yogaarir@gmail.com"
                                git config user.name "yoga"
                                git add ${valuesPath}
                                git commit -m "chore: update image tag to ${params.branch} [skip ci]" || echo "No changes to commit"
                                git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/yogaarie/template.git master
                            """
                        }
                    }
                }
            }
        }

        stage('Apply Argo') {
            steps {
                sh 'kubectl apply -f infra-templates/argo/prod-v1.yaml'
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}