pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS = credentials('docker')
        REPOSITORY_PREFIX = 'ender0'
    }

    stages {
        stage('Preparation') {
            steps {
                script {
                    // Remarquea : Les commandes 'sudo' peuvent nécessiter une configuration supplémentaire pour fonctionner dans Jenkins
                    sh '''
                        sudo apt update
                        sudo apt install maven -y
                        curl -fsSL https://get.docker.com | sudo sh
                        sudo chmod 666 /var/run/docker.sock
                        sudo usermod -aG docker $USER
                    '''
                }
            }
        }

        stage('Clean Docker') {
            steps {
                script {
                    sh '''
                        docker stop $(docker ps -aq) || true
                        docker rm -f $(docker ps -aq) || true
                        docker rmi -f $(docker images -q) || true
                    '''
                }
            }
        }

        stage('Clone Repository') {
            steps {
                sh '''
                    rm -rf /app
                    git clone https://github.com/devops-petclinic-group/spring-petclinic-cloud.git /app
                '''
            }
        }

        stage('Build and Push Docker Images') {
            steps {
                script {
                    sh '''
                        cd /app
                        mvn clean install
                        find . -type f -name "*.jar" -exec chmod 755 {} \;
                        mvn spring-boot:build-image -Dmaven.test.skip=true -Pk8s -DREPOSITORY_PREFIX=${REPOSITORY_PREFIX}
                        ./scripts/pushImages.sh
                    '''
                }
            }
        }

        stage('Deploy Services') {
            steps {
                sh '''
                    cd /app
                    docker-compose up -d
                '''
            }
        }

        stage('Run Selenium Tests and Update Report') {
            steps {
                sh '''
                    pytest --html=report.html > /reports/selenium_tests.txt
                    cd /reports
                    git pull origin master
                    git add selenium_tests.txt
                    git commit -m "Update Selenium test results"
                    git push origin master
                '''
            }
        }
    }

    post {
        always {
            // Nettoyage final, si nécessaire
        }
    }
}
