pipeline {
    agent any
    
    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Image tag to deploy')
    }
    
    environment {
        GIT_REPO = 'https://github.com/rimzoghlami/formationApp-k8s.git'
        BRANCH = 'main'
        DOCKER_USER = 'rimzoghlami'
        APP_NAME = 'cicd-pipeline'
    }
    
    stages {
        stage('Clone GitOps Repo') {
            steps {
                cleanWs()
                git branch: "${BRANCH}", credentialsId: 'github', url: "${GIT_REPO}"
            }
        }
        
        stage('Update Deployment Files') {
            steps {
                script {
                    def deploymentFiles = [
                        'mysql-deploy.yaml',
                        'eureka-server-deploy.yaml', 
                        'formation-service-deploy.yaml',
                        'user-service-deploy.yaml',
                        'frontend-deploy.yaml'
                    ]
                    
                    deploymentFiles.each { file ->
                        if (fileExists(file)) {
                            echo "Updating image tag in ${file} to ${params.IMAGE_TAG}"
                            
                            switch(file) {
                                case 'mysql-deploy.yaml':
                                    // MySQL typically uses stable tags, but you can update if needed
                                    sh """
                                        echo "MySQL deployment found - keeping stable mysql:8.0 image"
                                        # sed -i 's|image: mysql:.*|image: mysql:8.0|' ${file}
                                    """
                                    break
                                    
                                case 'eureka-server-deploy.yaml':
                                    sh """
                                        sed -i 's|image: ${DOCKER_USER}/${APP_NAME}-eureka-server:.*|image: ${DOCKER_USER}/${APP_NAME}-eureka-server:${params.IMAGE_TAG}|g' ${file}
                                    """
                                    break
                                    
                                case 'formation-service-deploy.yaml':
                                    sh """
                                        sed -i 's|image: ${DOCKER_USER}/${APP_NAME}-formation-service:.*|image: ${DOCKER_USER}/${APP_NAME}-formation-service:${params.IMAGE_TAG}|g' ${file}
                                    """
                                    break
                                    
                                case 'user-service-deploy.yaml':
                                    sh """
                                        sed -i 's|image: ${DOCKER_USER}/${APP_NAME}-user-service:.*|image: ${DOCKER_USER}/${APP_NAME}-user-service:${params.IMAGE_TAG}|g' ${file}
                                    """
                                    break
                                    
                                case 'frontend-deploy.yaml':
                                    sh """
                                        sed -i 's|image: ${DOCKER_USER}/${APP_NAME}-frontend:.*|image: ${DOCKER_USER}/${APP_NAME}-frontend:${params.IMAGE_TAG}|g' ${file}
                                    """
                                    break
                                    
                                default:
                                    echo "Unknown deployment file: ${file}"
                            }
                        } else {
                            echo "Warning: Deployment file not found: ${file}"
                        }
                    }
                }
            }
        }
        
        stage('Verify Changes') {
            steps {
                script {
                    sh '''
                        echo "=== Deployment files after update ==="
                        for file in *.yaml; do
                            if [ -f "$file" ]; then
                                echo "--- $file ---"
                                grep -n "image:" "$file" || echo "No image lines found in $file"
                                echo ""
                            fi
                        done
                    '''
                }
            }
        }
        
        stage('Commit & Push Changes') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    sh """
                        git config --global user.name "rimzoghlami"
                        git config --global user.email "rimzoghlami@esprit.tn"
                        
                        # Check if there are any changes
                        if git diff --quiet; then
                            echo "No changes detected in deployment files"
                            exit 0
                        fi
                        
                        # Add all yaml files that might have been updated
                        git add *.yaml || true
                        
                        # Commit with a descriptive message
                        git commit -m "Update deployment images to tag: ${params.IMAGE_TAG}
                        
                        - Updated formation-service image
                        - Updated user-service image  
                        - Updated eureka-server image
                        - Updated frontend image
                        
                        Triggered by CI build #${BUILD_NUMBER}"
                        
                        # Push changes
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/rimzoghlami/formationApp-k8s.git ${BRANCH}
                        
                        echo "Successfully pushed deployment updates to GitOps repository"
                    """
                }
            }
        }
        
        stage('Verify GitOps Update') {
            steps {
                script {
                    sh """
                        echo "=== GitOps Repository Update Summary ==="
                        echo "Repository: ${GIT_REPO}"
                        echo "Branch: ${BRANCH}"
                        echo "Image Tag: ${params.IMAGE_TAG}"
                        echo "Build Number: ${BUILD_NUMBER}"
                        echo ""
                        echo "Updated deployment files with new image tags."
                        echo "ArgoCD should detect these changes and trigger deployment."
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo """
            ✅ CD Pipeline completed successfully!
            
            📋 Summary:
            - GitOps repository updated: ${GIT_REPO}
            - Image tag deployed: ${params.IMAGE_TAG}
            - All deployment files updated with new images
            - Changes committed and pushed to ${BRANCH}
            
            🚀 Next Steps:
            - ArgoCD will detect the changes
            - Kubernetes deployments will be updated automatically
            - Monitor your cluster for the rollout status
            """
        }
        
        failure {
            echo """
            ❌ CD Pipeline failed!
            
            🔍 Troubleshooting:
            - Check if the GitOps repository exists: ${GIT_REPO}
            - Verify GitHub credentials are configured correctly
            - Ensure deployment YAML files exist in the repository
            - Check if the image tag ${params.IMAGE_TAG} is valid
            
            📋 Review the logs above for specific error details.
            """
        }
        
        always {
            cleanWs()
        }
    }
}
