stage('Run Ansible Liquibase Update') {
            when {
                expression { params.LIQUIBASE_STATUS == "1" }
            }
            steps {
                sh """
                    ansible-playbook liquibase-update.yml \
                        -e MYSQL_HOST=${params.MYSQL_HOST} \
                        -e MYSQL_PORT=${params.MYSQL_PORT} \
                        -e MYSQL_USER=${params.MYSQL_USER} \
                        -e MYSQL_PASSWORD=${params.MYSQL_PASSWORD} \
                        -e LIQUIBASE_SELECTED_CLIENTS=${params.LIQUIBASE_SELECTED_CLIENTS} \
                        -e LIQUIBASE_STATUS=${params.LIQUIBASE_STATUS} \
                        -e ENVIRONMENT=${params.ENVIRONMENT} \
                        -e DATABASES=${params.DATABASES}
                """
            }
        }
stage('Update GitOps Repository') {
    agent { label 'master' }
    steps {
        script {
            def gitToken = 'xxxxxxxxxxxxx'
            def newImage = "${REGISTRY_NAME}.azurecr.io/${JOB_NAME}.${ENVIRONMENT}:${BUILD_NUMBER}"
            
            sh """
                rm -rf gitops
                git clone -b qa1-alz https://jenkins:${gitToken}@git.nexquare.io/devops/gitops.git gitops
            """
            
            dir('gitops/liquibase') {
                sh """
                    echo "New image: ${newImage}"
                    
                    sed -i "s|image:.*|image: ${newImage}|g" liquibase.yml
                    
                    echo "Image updated successfully!"
                    grep "image:" liquibase.yml
                """
            }
            
            dir('gitops') {
                sh """
                    git -c user.name="jenkins" -c user.email="jenkins@nexquare.io" add liquibase/liquibase.yml
                    git -c user.name="jenkins" -c user.email="jenkins@nexquare.io" commit -m "Update image - Build #${BUILD_NUMBER}"
                    git -c user.name="jenkins" -c user.email="jenkins@nexquare.io" push https://jenkins:${gitToken}@git.nexquare.io/devops/gitops.git qa1-alz
                """
            }
        }
    }
}


    