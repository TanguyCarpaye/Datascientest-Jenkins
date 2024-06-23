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

        // stage('Container cleaning') {
        //     steps {
        //         sh '''
        //         # Enlever les conteneur utilisant le port 8080
        //         docker ps -a | grep '8080->8080' | awk '{print \$1}' | xargs --no-run-if-empty docker stop
        //         docker ps -a | grep '8080->8080' | awk '{print \$1}' | xargs --no-run-if-empty docker rm
        //         # Enlever le conteneur se nommant 'movie-service'
        //         docker ps -a | grep movie-service | awk '{print \$1}' | xargs --no-run-if-empty docker stop
        //         docker rm -f movie-service || true
        //         # Enlever le conteneur se nommant 'cast-service'
        //         docker ps -a | grep cast-service | awk '{print \$1}' | xargs --no-run-if-empty docker stop
        //         docker rm -f cast-service || true
        //         # Enlever le conteneur se nommant 'nginx'
        //         docker ps -a | grep nginx | awk '{print \$1}' | xargs --no-run-if-empty docker stop
        //         docker rm -f nginx || true
        //         '''
        //         }
        //     }
        
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

        
        stage('Convert Docker Compose to Kubernetes Configs') {
            steps {
                script {
                    sh 'kompose convert -f docker-compose.yml'
                }
            }
        }

        
        stage('Move Kubernetes Configs') {
        steps {
            script {
                sh """
                # Création des répertoires pour les microservices et les composants
                sudo mkdir -p /home/ubuntu/manifestsKubernetes/my-application/templates/cast
                sudo mkdir -p /home/ubuntu/manifestsKubernetes/my-application/templates/movie
                sudo mkdir -p /home/ubuntu/manifestsKubernetes/my-application/templates/nginx
                sudo mkdir -p /home/ubuntu/manifestsKubernetes/my-application/templates/postgres
                # Déplacement des fichiers pour le microservice "Cast"
                sudo mv cast-service-claim0-persistentvolumeclaim.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/cast/cast-service-claim0-persistentvolumeclaim.yaml
                sudo mv cast-service-deployment.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/cast/cast-service-deployment.yaml
                sudo mv cast_service-service.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/cast/cast-service-service.yaml
                sudo sed -i 's/name: cast_db/name: cast-db/g' /home/ubuntu/manifestsKubernetes/my-application/templates/cast/cast-db-deployment.yaml
                sudo sed -i 's/name: cast_db/name: cast-db/g' /home/ubuntu/manifestsKubernetes/my-application/templates/cast/cast-service-deployment.yaml
                # Déplacement des fichiers pour le microservice "Movie"
                sudo mv movie-db-deployment.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/movie/movie-db-deployment.yaml
                sudo mv movie-service-claim0-persistentvolumeclaim.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/movie/movie-service-claim0-persistentvolumeclaim.yaml
                sudo mv movie-service-deployment.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/movie/movie-service-deployment.yaml
                sudo mv movie_service-service.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/movie/movie-service-service.yaml
                # Déplacement des fichiers pour Nginx
                sudo mv nginx-claim0-persistentvolumeclaim.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/nginx/nginx-claim0-persistentvolumeclaim.yaml
                sudo mv nginx-deployment.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/nginx/nginx-deployment.yaml
                sudo mv nginx-service.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/nginx/nginx-service.yaml
                # Déplacement des fichiers pour les données PostgreSQL
                sudo mv postgres-data-cast-persistentvolumeclaim.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/postgres/postgres-data-cast-persistentvolumeclaim.yaml
                """
                sh 'ls'
            }
        }
    }

    stage('Correction de l orthographe des fichiers') {
    steps {
        script {
            sh 'ls'
            sh 'mv cast_db-service.yaml cast-db-service.yaml'
            sh 'mv movie_db-service.yaml movie-db-service.yaml'
            sh 'mv nginx_config.conf nginx-config.conf'
            sh 'ls'
            }
        }
    }
        
    stage('Test Kubectl Connectivity') {
    steps {
        script {
            sh 'kubectl version'
            sh 'kubectl get nodes'
            sh 'kubectl cluster-info'
            sh 'touch test.txt'
            }
        }
    }

    stage('Test Helm Connectivity') {
    steps {
        script {
            sh 'helm version'
            sh 'ls'
            }
        }
    }
    
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
