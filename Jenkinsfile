pipeline {
    agent any

    tools {
        jdk 'jdk17'          // Jenkins JDK tool name
        maven 'maven3'       // Jenkins Maven tool name
        // Sonar scanner tool is configured later in stages
    }

    environment {
        // SonarQube
        SONARQUBE_SERVER = 'sonarqube-server'
        SONARQUBE_CREDENTIALS = 'sonar-token'  // Secret Text credential ID in Jenkins

        // JFrog Artifactory
        JFROG_CREDENTIALS = 'jfrog-creds'      // Username + API token in Jenkins
        JFROG_URL = 'https://trialepv7i1.jfrog.io/artifactory/petclinic-repo/'

        // Docker
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
                sh 'mvn clean install'
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
                    sh "mvn sonar:sonar -Dsonar.login=${SONARQUBE_CREDENTIALS}"
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
                sh "trivy image --exit-code 1 --severity HIGH,CRITICAL ${DOCKER_IMAGE}:latest || true"
            }
        }

        stage('Push to JFrog') {
            steps {
                script {
                    sh """
                        docker login ${JFROG_URL} -u ${JFROG_CREDENTIALS_USR} -p ${JFROG_CREDENTIALS_PSW}
                        docker tag ${DOCKER_IMAGE}:latest ${JFROG_URL}${DOCKER_IMAGE}:latest
                        docker push ${JFROG_URL}${DOCKER_IMAGE}:latest
                    """
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
