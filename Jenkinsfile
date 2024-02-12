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
                    try {
                        echo 'Mise à jour du système et configuration Docker...'
                        sh '''
                            set -x
                            sudo apt update
                            sudo chmod 666 /var/run/docker.sock
                            sudo usermod -aG docker $USER
                        '''
                    } catch (Exception e) {
                        echo "Erreur lors de la préparation : ${e.getMessage()}"
                    }
                }
            }
        }

        stage('Setup /app Directory') {
            steps {
                script {
                    try {
                        echo 'Configuration du répertoire /app...'
                        sh '''
                            set -x
                            sudo mkdir -p /app
                            sudo chown $USER:$USER /app
                            sudo chmod 777 /app
                        '''
                    } catch (Exception e) {
                        echo "Erreur lors de la configuration du répertoire /app : ${e.getMessage()}"
                    }
                }
            }
        }

        stage('Clean Docker') {
            steps {
                script {
                    try {
                        echo 'Nettoyage des conteneurs et images Docker...'
                        sh '''
                            set -x
                            docker stop $(docker ps -aq) || true
                            docker rm -f $(docker ps -aq) || true
                            docker rmi -f $(docker images -q) || true
                        '''
                    } catch (Exception e) {
                        echo "Erreur lors du nettoyage Docker : ${e.getMessage()}"
                    }
                }
            }
        }

        stage('Clone Repository') {
            steps {
                script {
                    try {
                        echo 'Clonage du dépôt Git...'
                        sh '''
                            set -x
                            git clone https://github.com/devops-petclinic-group/spring-petclinic-cloud.git /app
                        '''
                    } catch (Exception e) {
                        echo "Erreur lors du clonage du dépôt : ${e.getMessage()}"
                    }
                }
            }
        }

        stage('Build and Push Docker Images') {
            steps {
                script {
                    try {
                        echo 'Construction et déploiement des images Docker...'
                        sh '''
                            set -x
                            cd /app
                            mvn clean install
                            find . -type f -name "*.jar" -exec chmod 755 {} \\;
                            mvn spring-boot:build-image -Dmaven.test.skip=true -Pk8s -DREPOSITORY_PREFIX=${REPOSITORY_PREFIX}
                            ./scripts/pushImages.sh
                        '''
                    } catch (Exception e) {
                        echo "Erreur lors de la construction et du déploiement : ${e.getMessage()}"
                    }
                }
            }
        }

        stage('Deploy Services') {
            steps {
                script {
                    try {
                        echo 'Déploiement des services...'
                        sh '''
                            set -x
                            cd /app
                            docker-compose up -d
                        '''
                    } catch (Exception e) {
                        echo "Erreur lors du déploiement des services : ${e.getMessage()}"
                    }
                }
            }
        }

        stage('Run Selenium Tests and Update Report') {
            steps {
                script {
                    try {
                        echo 'Exécution des tests Selenium et mise à jour du rapport...'
                        sh '''
                            set -x
                            pytest --html=report.html > /reports/selenium_tests.txt
                            cd /reports
                            git pull origin master
                            git add selenium_tests.txt
                            git commit -m "Update Selenium test results"
                            git push origin master
                        '''
                    } catch (Exception e) {
                        echo "Erreur lors des tests Selenium : ${e.getMessage()}"
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Le processus est terminé"
        }
    }
}
