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
                sudo mkdir -p /home/ubuntu/manifestsKubernetes/my-application/templates
                sudo mv cast-db-deployment.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/cast-db-deployment.yaml
                sudo mv cast-service-claim0-persistentvolumeclaim.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/cast-service-claim0-persistentvolumeclaim.yaml
                sudo mv cast-service-deployment.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/cast-service-deployment.yaml
                sudo mv cast_service-service.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/cast-service-service.yaml
                sudo mv movie-db-deployment.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/movie-db-deployment.yaml
                sudo mv movie-service-claim0-persistentvolumeclaim.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/movie-service-claim0-persistentvolumeclaim.yaml
                sudo mv movie-service-deployment.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/movie-service-deployment.yaml
                sudo mv movie_service-service.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/movie-service-service.yaml
                sudo mv nginx-claim0-persistentvolumeclaim.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/nginx-claim0-persistentvolumeclaim.yaml
                sudo mv nginx-deployment.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/nginx-deployment.yaml
                sudo mv nginx-service.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/nginx-service.yaml
                sudo mv postgres-data-cast-persistentvolumeclaim.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/postgres-data-cast-persistentvolumeclaim.yaml
                sudo mv postgres-data-movie-persistentvolumeclaim.yaml /home/ubuntu/manifestsKubernetes/my-application/templates/postgres-data-movie-persistentvolumeclaim.yaml
                sudo sed -i 's/name: cast_db/name: cast-db/g' /home/ubuntu/manifestsKubernetes/my-application/templates/cast-db-deployment.yaml
                sudo sed -i 's/name: cast_db/name: cast-db/g' /home/ubuntu/manifestsKubernetes/my-application/templates/cast-service-deployment.yaml
                """
            }
        }
    }

    stage('Test Kubectl Connectivity') {
    steps {
        script {
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

    stage('Set Kubectl Context') {
    steps {
        script {
            sh 'sudo kubectl config use-context default'
            }
        }
    }
    
    stage('Deploy to Kubernetes with Helm') {
        steps {
            script {
                def environments = ['dev', 'qa', 'staging', 'prod']
                environments.each {
                    env -> 
                    sh """
                    helm upgrade --install my-application-release /home/ubuntu/manifestsKubernetes/my-application --namespace=${env}
                    """
                }
            }
        }
        }
    }
}
