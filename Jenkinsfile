pipeline {
environment {  // Declaration of environment variables
DOCKER_ID = "foker84"
DOCKER_IMAGE_POSTGRES = "postgres"
DOCKER_TAG_POSTGRE = "12.1-alpine"
DOCKER_IMAGE_NGINX = "nginx"
DOCKER_TAG_NGINX = "latest"
DOCKER_IMAGE_CAST = "jenkins_devops_exams-cast_service"
DOCKER_IMAGE_MOVIE = "jenkins_devops_exams-movie_service"
DOCKER_TAG_1 = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages {
        stage(' Docker Build image'){ // docker build different image
            steps {
               dir('movie-service'){
               script {
                sh '''
                 #docker rm -f jenkins
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE_NGINX:$DOCKER_TAG_NGINX .
                 sleep 10
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG_1 .			
                '''
                }}
               dir('cast-service'){
               script {
                sh '''
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG_1 .
                '''
                }}
            }
        }
        stage('Docker run'){ // run container from our builded image cast
                steps {
                    script {
                    sh '''
                    #docker run --name postgrejenkins $DOCKER_IMAGE_POSTGRES:$DOCKER_TAG_POSTGRE
                    docker run -d --name nginxjenkins2 $DOCKER_IMAGE_NGINX:$DOCKER_TAG_NGINX
		    sleep 6
		     docker run -d --name castpostgre_container2 -e postgres_user=cast_db_username -e postgres_password=cast_db_password -e postgres_db=cast_db_dev postgres:12.1-alpine
		    #sleep 10
		     #docker run --name moviepostgre_container -e postgres_user=movie_db_username -e postgres_password=movie_db_password -e postgres_db=movie_db_dev postgres:12.1-alpine
		    sleep 10
                     docker run -d -p 8001:8000 --name moviejenkins3 $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG_1
                    sleep 10
                     docker run -d -p 8002:8000 --name castjenkins3 $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG_1
                    '''
                    }
                }
            }

        stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    curl localhost
                    '''
                    }
            }

        }
        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_IMAGE_NGINX:$DOCKER_TAG_NGINX
                docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG_1
                docker push $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG_1
                '''
                }
            }

        }

stage('Deploiement en dev'){
        environment
        {
		KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp examhelm/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG_1}+g" values.yml
		helm upgrade --install examhelm ./examhelm/ --values=./examhelm/values.yaml --namespace dev
                '''
                }
            }

        }
stage('Deploiement en staging'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp examhelm/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG_1}+g" values.yml
                helm upgrade --install examhelm ./examhelm/ --values=./examhelm/values.yaml --namespace staging
                '''
                }
            }
  }
  stage('Deploiement en prod'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
            // Create an Approval Button with a timeout of only 15minutes.
            // this require a manuel validation in order to deploy on production environment
                    timeout(time: 15, unit: "MINUTES") {
                        input message: 'Do you want to deploy in production ?', ok: 'Yes'
                    }

                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp examhelm/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG_1}+g" values.yml
		helm upgrade --install examhelm ./examhelm/ --values=./examhelm/values.yaml --namespace prod
                '''
                }
            }

        }

}
}


