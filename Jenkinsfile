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
            }
        }
    }
    stage('Test Kubectl Connectivity') {
    steps {
        script {
            sh 'kubectl version'
            sh 'kubectl get nodes'
            sh 'kubectl cluster-info'
            }
        }
    }
    stage('Test Helm Connectivity') {
    steps {
        script {
            sh 'helm version'
            }
        }
    }
//    stage('Set Kubectl Context') {

//    steps {

//        script {

//            sh 'sudo kubectl config use-context default'

//            sh 'kubectl config use-context default'

//            }

//        }

//    }
    

//    stage('Deploy to Kubernetes with Helm') {

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
    }
}
