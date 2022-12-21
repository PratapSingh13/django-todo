node
{
    //agent any
    office365ConnectorSend color: '#f39442', \
        message: 'Todo-CI Jenkins job started', \
        status: 'Started', \
        webhookUrl: 'https://coredgeio.webhook.office.com/webhookb2/fd10decb-b1c3-4fe7-bd86-065a39b72f5f@6fc131bd-72da-4e55-bb85-5e873ab21dc6/JenkinsCI/c741708870da4245a68ca21cc0891940/620917ee-71b2-4d2d-9a1c-aa22aa637ef1'
    cleanWs()
    try
    {
        stage("Checkout Source Code")
        {
            try
            {
                git branch: 'main', credentialsId: 'pratap-github-coredge', url: 'https://github.com/PratapSingh13/django-todo.git'
            }
            catch(Exception e)
            {
                echo "Code Checkout Failed"
                office365ConnectorSend color: '#fb0400', \
                    message: 'Code Checkout Failed', \
                    status: 'Failed', \
                    webhookUrl: 'https://coredgeio.webhook.office.com/webhookb2/fd10decb-b1c3-4fe7-bd86-065a39b72f5f@6fc131bd-72da-4e55-bb85-5e873ab21dc6/JenkinsCI/c741708870da4245a68ca21cc0891940/620917ee-71b2-4d2d-9a1c-aa22aa637ef1'
                echo e.toString()
                throw e
            }
        }
        stage("Dependency Check")
        {
            try
            {
                dependencyCheck additionalArguments: '--format HTML', odcInstallation: 'OWASP-Dependency-Check'
            }
            catch(Exception e)
            {
                echo "Dependency check failed"
                office365ConnectorSend color: '#fb0400', \
                    message: 'Dependency check failed', \
                    status: 'Failed', \
                    webhookUrl: 'https://coredgeio.webhook.office.com/webhookb2/fd10decb-b1c3-4fe7-bd86-065a39b72f5f@6fc131bd-72da-4e55-bb85-5e873ab21dc6/JenkinsCI/c741708870da4245a68ca21cc0891940/620917ee-71b2-4d2d-9a1c-aa22aa637ef1'
                echo e.toString()
                throw e
            }
        }
        stage("Repository Scan")
        {
            sh 'trivy fs .'
        }
        stage("Build Dockerfile")
        {
            try
            {
                //sh 'docker build --no-cache  --platform linux/amd64 -t pratapsingh13/todo-app:latest .'
                
                def tag = sh(script: '''echo pratapsingh13/todo-app
                git log -1 --pretty=%h
                date +%Y-%m-%d-%H-%M-%S''', returnStdout: true)
                //def hashId = sh(script: '''git log -1 --pretty=%h''', returnStdout: true)
                //def tag = sh(script: '''${timestamp}-${hashId}''', returnStdout: true)
                //echo "${timestamp}"
                //echo "${hashId}"
                echo "${tag}"
                dockerImage = docker.build("pratapsingh13/todo-app:latest")
            }
            catch(Exception e)
            {
                echo "Docker build failed"
                office365ConnectorSend color: '#fb0400', \
                    message: 'Docker build failed', \
                    status: 'Failed', \
                    webhookUrl: 'https://coredgeio.webhook.office.com/webhookb2/fd10decb-b1c3-4fe7-bd86-065a39b72f5f@6fc131bd-72da-4e55-bb85-5e873ab21dc6/JenkinsCI/c741708870da4245a68ca21cc0891940/620917ee-71b2-4d2d-9a1c-aa22aa637ef1'
                echo e.toString()
                throw e
            }
        }
        stage("Scan Docker Image")
        {
            try
            {
                echo "Image Scanning..."
                sh 'trivy image --reset'
                sh 'trivy image --format template --template "@/home/core/data/contrib/html.tpl" --exit-code 1 --severity HIGH,CRITICAL -o docker_image_scan_report.html pratapsingh13/todo-app:latest'
                publishHTML([
                    allowMissing: false, 
                    alwaysLinkToLastBuild: false, 
                    keepAll: false, 
                    reportDir: '', 
                    reportFiles: 'docker_image_scan_report.html', 
                    reportName: 'docker_image_scan_report.html', 
                    reportTitles: '', 
                    useWrapperFileDirectly: true
                ])
            }
            catch(Exception e)
            {
                echo "Image scanning failed"
                office365ConnectorSend color: '#fb0400', \
                    message: 'Image scanning failed', \
                    status: 'Failed', \
                    webhookUrl: 'https://coredgeio.webhook.office.com/webhookb2/fd10decb-b1c3-4fe7-bd86-065a39b72f5f@6fc131bd-72da-4e55-bb85-5e873ab21dc6/JenkinsCI/c741708870da4245a68ca21cc0891940/620917ee-71b2-4d2d-9a1c-aa22aa637ef1'
                echo e.toString()
                throw e
            }
        }
        stage("Login to Docker Hub")
        {
            try
            {
                echo "Making connection for login to DockerHub"
                //sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                withDockerRegistry([credentialsId: 'pratap-dockerhub-account', url: ""])
                {
                    stage("Upload Image to DockerHub")
                    {
                        try
                        {
                            echo "Trying to Image Push"
                            sh 'docker tag pratapsingh13/todo-app:latest pratapsingh13/todo-app:${BUILD_TIMESTAMP}-${BUILD_NUMBER}'
                            dockerImage.push()
                        }
                        catch(Exception e)
                        {
                            echo "Image uplaod failed"
                            office365ConnectorSend color: '#fb0400', \
                                message: 'Failed to upload image on DockerHub', \
                                status: 'Failed', \
                                webhookUrl: 'https://coredgeio.webhook.office.com/webhookb2/fd10decb-b1c3-4fe7-bd86-065a39b72f5f@6fc131bd-72da-4e55-bb85-5e873ab21dc6/JenkinsCI/c741708870da4245a68ca21cc0891940/620917ee-71b2-4d2d-9a1c-aa22aa637ef1'
                            echo e.toString()
                            throw e
                        }
                    }
                }
            }
            catch(Exception e)
            {
                echo "Docker login failed"
                office365ConnectorSend color: '#fb0400', \
                    message: 'Failed to Login into Docker', \
                    status: 'Failed', \
                    webhookUrl: 'https://coredgeio.webhook.office.com/webhookb2/fd10decb-b1c3-4fe7-bd86-065a39b72f5f@6fc131bd-72da-4e55-bb85-5e873ab21dc6/JenkinsCI/c741708870da4245a68ca21cc0891940/620917ee-71b2-4d2d-9a1c-aa22aa637ef1'
                echo e.toString()
                throw e
            }
        }
        stage("Remove unused Image")
        {
            try
            {
                sh 'docker image prune -a -f'
            }
            catch(Exception e)
            {
                echo "Unused image deletion failed"
                office365ConnectorSend color: '#fb0400', \
                    message: 'Unused image deletion failed', \
                    tatus: 'Failed', \
                    webhookUrl: 'https://coredgeio.webhook.office.com/webhookb2/fd10decb-b1c3-4fe7-bd86-065a39b72f5f@6fc131bd-72da-4e55-bb85-5e873ab21dc6/JenkinsCI/c741708870da4245a68ca21cc0891940/620917ee-71b2-4d2d-9a1c-aa22aa637ef1'
                echo e.toString()
                throw e
            }
        }
        stage("Sending Notification")
        {
            office365ConnectorSend color: '#00a52a', \
                message: 'Job successfully executed', \
                status: 'Success', \
                webhookUrl: 'https://coredgeio.webhook.office.com/webhookb2/fd10decb-b1c3-4fe7-bd86-065a39b72f5f@6fc131bd-72da-4e55-bb85-5e873ab21dc6/JenkinsCI/c741708870da4245a68ca21cc0891940/620917ee-71b2-4d2d-9a1c-aa22aa637ef1'
        }
    }
    catch(Exception e)
    {
        echo "Pipeline failed please check once again"
        office365ConnectorSend color: '#fb0400', \
            message: 'Pipeline failed', \
            status: 'Failed', \
            webhookUrl: 'https://coredgeio.webhook.office.com/webhookb2/fd10decb-b1c3-4fe7-bd86-065a39b72f5f@6fc131bd-72da-4e55-bb85-5e873ab21dc6/JenkinsCI/c741708870da4245a68ca21cc0891940/620917ee-71b2-4d2d-9a1c-aa22aa637ef1'
        echo e.toString()
        throw e
    }
}
