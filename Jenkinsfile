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
                            mkdir -p $HOME/bin; \
                            curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o $HOME/bin/docker-compose; \
                            chmod +x $HOME/bin/docker-compose; \
                            export PATH=$PATH:$HOME/bin; \
                        fi'
                }
            }
        }

        stage('Building and Running Docker Compose') {
            steps {
                script {
                    sh '/var/lib/jenkins/bin/docker-compose up -d'
                }
            }
        }
    }
}
