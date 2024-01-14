pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_VERSION = '1.29.2'
        KUBE_CONFIG = '/path/to/your/kubeconfig'
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
                    if ! [ -x "$(command -v docker-compose)" ]; then
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
                    // Assurez-vous que Kompose est installé
                    sh 'kompose convert -f docker-compose.yml'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Modifier 'dev', 'qa', 'staging', 'prod' selon vos namespaces
                    def environments = ['dev', 'qa', 'staging', 'prod']
                    environments.each {
                        env -> 
                        sh """
                        kubectl apply -f <your-kubernetes-configs-directory> --namespace=${env}
                        """
                    }
                }
            }
        }
    }
}
