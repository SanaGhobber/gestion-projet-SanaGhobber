pipeline {
    agent any

    tools {
        jdk 'JDK17'
    }

    environment {
        DOCKER_IMAGE = 'ghobberrr/gestion-projet-backend'
        DOCKER_TAG = '1.0'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/SanaGhobber/gestion-projet-SanaGhobber.git'
            }
        }

        stage('Build') {
            steps {
                dir('backend') {
                    sh 'chmod +x mvnw'
                    sh './mvnw clean package -DskipTests'
                }
            }
        }

        stage('Tests') {
           steps {
              withEnv(['DB_PASSWORD=YOUR_PASSWORD']) {
                dir('backend') {
                   sh './mvnw test'
                  }
              }
           }
       }

        stage('SonarQube Analysis') {
            steps {
                dir('backend') {
                    withSonarQubeEnv('SonarQube') {
                        sh './mvnw sonar:sonar -Dsonar.projectKey=gestion-projet-backend'
                    }
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir('backend') {
                    sh 'docker build -t $DOCKER_IMAGE:$DOCKER_TAG .'
                }
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKER_IMAGE:$DOCKER_TAG
                        docker logout
                    '''
                }
            }
        }
    }
}
