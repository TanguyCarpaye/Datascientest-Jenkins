pipeline {
    agent any

    stages {
        stage('Cloning Git Repository') {
            steps {
                git 'https://github.com/TanguyCarpaye/Datascientest-Jenkins'
            }
        }

        stage('Install Docker Compose') {
            steps {
                script {
                    sh 'if ! [ -x "$(command -v docker-compose)" ]; then \
                            sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose; \
                            sudo chmod +x /usr/local/bin/docker-compose; \
                        fi'
                }
            }
        }

        stage('Building and Running Docker Compose') {
            steps {
                script {
                    sh 'docker-compose up -d'
                }
            }
        }
    }
}
