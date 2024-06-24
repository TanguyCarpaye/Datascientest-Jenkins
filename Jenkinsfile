pipeline {
    agent any
    environment {
        DOCKER_COMPOSE_VERSION = '1.29.2'
        KUBE_CONFIG = '/home/ubuntu/.kube/config'
    }
    stages {
        stage('Cloning Git Repository') {
            steps {
                git 'https://github.com/TanguyCarpaye/Datascientest-Jenkins'
            }
        }

        stage('Docker Hub Connection') {
            steps {
                withCredentials([usernamePassword(credentialsId: '4828fe9d-b6b7-4045-8561-036147dfcf52', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                      sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                  }
                }
            }

        
        stage('Docker Push'){ //we pass the built image to our docker hub account
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: '4828fe9d-b6b7-4045-8561-036147dfcf52', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    dir('movie-service') {
                    sh '''
                    # DockerHub connection
                    echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                    # movie-service
                    docker build -t my-registry/movie-service:movie-service .
                    docker tag my-registry/movie-service:movie-service tanguycarpaye/jenkinsdevopsexams:movie-service
                    docker push tanguycarpaye/jenkinsdevopsexams:movie-service
                    '''
                                        }
                    dir('cast-service') {
                    sh '''
                    # DockerHub connection
                    echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                    # cast-service
                    docker build -t my-registry/movie-service:cast-service .
                    docker tag my-registry/movie-service:cast-service tanguycarpaye/jenkinsdevopsexams:cast-service
                    docker push tanguycarpaye/jenkinsdevopsexams:cast-service
                    '''
                                        }
                    dir('nginx') {
                    sh '''
                    # DockerHub connection
                    echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                    # nginx
                    docker pull nginx:latest
                    docker tag nginx:latest tanguycarpaye/jenkinsdevopsexams:nginx
                    docker push tanguycarpaye/jenkinsdevopsexams:nginx
                    '''
                                        }
                        }
                    }
                }
            }

        
        stage('Install Docker Compose') {
            steps {
                script {
                    sh """
                    if ! [ -x \$(command -v docker-compose) ]; then
                        mkdir -p \$HOME/bin
                        curl -L "https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-\$(uname -s)-\$(uname -m)" -o \$HOME/bin/docker-compose
                        chmod +x \$HOME/bin/docker-compose
                        export PATH=\$PATH:\$HOME/bin
                    fi
                    """
                }
            }
        }


        stage('Test Kubectl and Helm Connectivity') {
            steps {
                script {
                sh 'kubectl version'
                sh 'kubectl get nodes'
                sh 'kubectl cluster-info'
                sh 'helm version'
                        }
                    }
                }
        

        stage('Setup Kubernetes Namespaces') {
            steps {
            script {
                // Créer le namespace 'dev' s'il n'existe pas
                sh 'kubectl get ns dev || kubectl create namespace dev'
                // Créer le namespace 'qa' s'il n'existe pas
                sh 'kubectl get ns qa || kubectl create namespace qa'
                // Créer le namespace 'staging' s'il n'existe pas
                sh 'kubectl get ns staging || kubectl create namespace staging'
                // Créer le namespace 'prod' s'il n'existe pas
                sh 'kubectl get ns prod || kubectl create namespace prod'
                }
            }
        }

        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh 'kompose convert -f docker-compose.yml -o k8s/'
                    // Renommer les services et les déploiements pour respecter les conventions Kubernetes
                    sh 'find k8s/ -type f -name "*.yaml" -exec sed -i "s/cast_service/cast-service/g" {} +'
                    sh 'find k8s/ -type f -name "*.yaml" -exec sed -i "s/movie_service/movie-service/g" {} +'
                    sh 'find k8s/ -type f -name "*.yaml" -exec sed -i "s/cast_db/cast-db/g" {} +'
                    sh 'find k8s/ -type f -name "*.yaml" -exec sed -i "s/movie_db/movie-db/g" {} +'
                    // Appliquer les configurations modifiées aux environnements spécifiques
                    sh 'kubectl apply -f k8s/ --namespace=dev'
                    sh 'kubectl apply -f k8s/ --namespace=qa'
                    sh 'kubectl apply -f k8s/ --namespace=staging'
                }
            }
        }
        
        
        stage('Charts Helm') {
            steps {
                script {
                    sh '''
                    sudo apt-get install tree
                    tree
                    cp -r helm_templates/movie-app helm/movie-app/
                    mv k8s/movie_service-deployment.yaml helm/movie-app/templates/
                    mv k8s/movie_service-service.yaml helm/movie-app/templates/
                    cp -r helm_templates/cast-app helm/cast-app/
                    mv k8s/cast_service-deployment.yaml helm/cast-app/templates/
                    mv k8s/cast_service-service.yaml helm/cast-app/templates/
                    '''
                }
            }
        }
        
        
        stage('Manual Deployment to Production') {
            when {
                branch 'master'
                 }
            steps {
                input 'Deploy to Production?'
                script {
                    sh 'helm upgrade --install movie-app helm/movie-app/ --namespace prod'
                       }
                  }
         }
        

        
        // stage('Convert Docker Compose to Kubernetes Configs') {
        //     steps {
        //         script {
        //             sh 'kompose convert -f docker-compose.yml'
        //         }
        //     }
        // }
        // stage('Move Kubernetes Configs') {
        // steps {
        //     script {
        //         sh """
        //         # Création des répertoires pour les microservices et les composants
        //         sudo mkdir -p /home/ubuntu/manifestsKubernetes/my-application/templates/cast
        //         sudo mkdir -p /home/ubuntu/manifestsKubernetes/my-application/templates/movie
        //         sudo mkdir -p /home/ubuntu/manifestsKubernetes/my-application/templates/nginx
        //         sudo mkdir -p /home/ubuntu/manifestsKubernetes/my-application/templates/postgres
        //         # Déplacement des fichiers pour le microservice "Cast"
        //         sudo mv cast-service-claim0-persistentvolumeclaim.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/cast/cast-service-claim0-persistentvolumeclaim.yaml
        //         sudo mv cast-service-deployment.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/cast/cast-service-deployment.yaml
        //         sudo mv cast_service-service.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/cast/cast-service-service.yaml
        //         sudo sed -i 's/name: cast_db/name: cast-db/g' /home/ubuntu/manifestsKubernetes/my-application/templates/cast/cast-db-deployment.yaml
        //         sudo sed -i 's/name: cast_db/name: cast-db/g' /home/ubuntu/manifestsKubernetes/my-application/templates/cast/cast-service-deployment.yaml
        //         # Déplacement des fichiers pour le microservice "Movie"
        //         sudo mv movie-db-deployment.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/movie/movie-db-deployment.yaml
        //         sudo mv movie-service-claim0-persistentvolumeclaim.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/movie/movie-service-claim0-persistentvolumeclaim.yaml
        //         sudo mv movie-service-deployment.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/movie/movie-service-deployment.yaml
        //         sudo mv movie_service-service.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/movie/movie-service-service.yaml
        //         # Déplacement des fichiers pour Nginx
        //         sudo mv nginx-claim0-persistentvolumeclaim.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/nginx/nginx-claim0-persistentvolumeclaim.yaml
        //         sudo mv nginx-deployment.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/nginx/nginx-deployment.yaml
        //         sudo mv nginx-service.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/nginx/nginx-service.yaml
        //         # Déplacement des fichiers pour les données PostgreSQL
        //         sudo mv postgres-data-cast-persistentvolumeclaim.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/postgres/postgres-data-cast-persistentvolumeclaim.yaml
        //         """
        //         sh 'ls'
        //     }
        //   }
        // }
        // stage('Correction de l orthographe des fichiers') {
        // steps {
        //     script {
        //         sh 'ls'
        //         sh 'mv cast_db-service.yaml cast-db-service.yaml'
        //         sh 'mv movie_db-service.yaml movie-db-service.yaml'
        //         sh 'mv nginx_config.conf nginx-config.conf'
        //         sh 'ls'
        //         }
        //       }
        //   }
        


    
//    stage('Deploy to Kubernetes with Helm') {
//        environment
//        {
//        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
//        }
//        steps {
//            script {
//                def environments = ['dev', 'qa', 'staging', 'prod']
//                environments.each {
//                    env -> 
//                    sh """
//                    helm upgrade --install my-application-release /home/ubuntu/manifestsKubernetes/my-application --namespace=${env}
//                    """
//                }
//            }
//        }
//        }
        
//    stage('Deploiement en dev'){
//        environment
//        {
//        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
//        }
//            steps {
//                script {
//                sh '''
//                rm -Rf .kube
//                mkdir .kube
//                ls
//                cat $KUBECONFIG > .kube/config
//                cp fastapi/values.yaml values.yml
//                cat values.yml
//                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
//                helm upgrade --install app fastapi --values=values.yml --namespace dev
//                '''
//                }
//            }
//        }


        
//    stage('Create Helm Chart') {
//    steps {
//        script {
//            sh """
//            # Création d'un nouveau chart Helm pour l'application
//            helm create my-application
//            # Copie des configurations Kubernetes dans le dossier templates du chart Helm
//            cp -r /home/ubuntu/manifestsKubernetes/my-application/templates/* my-application/templates/
//            """
//            sh 'ls'
//            }
//        }
//    }


        
//    stage('Deploy with Helm') {
//    steps {
//        script {
//            withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
//                sh """
//                mkdir -p .kube
//                cp $KUBECONFIG .kube/config
//                helm upgrade --install my-release my-application --namespace dev
//                """
//                }
//            }
//        }
//    }






        
//  stage('Deploiement en prod'){
//        environment
//        {
//        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
//        }
//            steps {
//            // Create an Approval Button with a timeout of 15minutes.
//            // this require a manuel validation in order to deploy on production environment
//                    timeout(time: 15, unit: "MINUTES") {
//                        input message: 'Do you want to deploy in production ?', ok: 'Yes'
//                    }
//                script {
//                sh '''
//                rm -Rf .kube
//                mkdir .kube
//                ls
//                cat $KUBECONFIG > .kube/config
//                cp fastapi/values.yaml values.yml
//                cat values.yml
//                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
//                helm upgrade --install app fastapi --values=values.yml --namespace prod
//                '''
//                }
//            }
//        }

        
        
        
        
    }
}
