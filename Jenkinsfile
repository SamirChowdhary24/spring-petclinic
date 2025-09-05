pipeline {
    agent any

    tools {
        jdk 'jdk17'          // Jenkins JDK tool name
        maven 'maven3'       // Jenkins Maven tool name
        // Sonar scanner tool is configured later in stages
    }

    environment {
        SONARQUBE_SERVER = 'sonarqube-server'
        SONARQUBE_CREDENTIALS = 'sonar-token'
    
        JFROG_URL = 'https://trialepv7i1.jfrog.io/artifactory/petclinic-repo/'
    
        DOCKER_IMAGE = 'petclinic-app'
        DOCKER_CONTAINER_PORT = '8082'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/SamirChowdhary24/spring-petclinic.git'
            }
        }

        stage('Build & Test') {
             steps {
                // Run only MySQL integration tests
                sh '''
                    mvn clean test \
                    -Dspring.profiles.active=mysql \
                    -Dtest=**/*MySqlIntegrationTests.java \
                    -DskipTests=false
                '''
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:latest ."
            }
        }

        stage('Trivy Scan') {
            steps {
                sh "trivy image --exit-code 1 --severity HIGH,CRITICAL ${DOCKER_IMAGE}:latest "
            }
        }

        stage('Push to JFrog') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'jfrog-creds', 
                                                     usernameVariable: 'JFROG_USER', 
                                                     passwordVariable: 'JFROG_PASSWORD')]) {
                        sh """
                            docker login ${JFROG_URL} -u ${JFROG_USER} -p ${JFROG_PASSWORD}
                            docker tag ${DOCKER_IMAGE}:latest ${JFROG_URL}${DOCKER_IMAGE}:latest
                            docker push ${JFROG_URL}${DOCKER_IMAGE}:latest
                        """
                    } 
                } 
            }
        } 


        stage('Deploy Container') {
            steps {
                sh """
                    docker stop ${DOCKER_IMAGE} || true
                    docker rm ${DOCKER_IMAGE} || true
                    docker run -d --name ${DOCKER_IMAGE} -p ${DOCKER_CONTAINER_PORT}:8080 ${DOCKER_IMAGE}:latest
                """
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
    }
}
