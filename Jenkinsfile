pipeline {
    agent any

    tools {
        jdk 'jdk17'        // Jenkins JDK tool name
        maven 'maven3'     // Jenkins Maven tool name
    }

    environment {
        // SONARQUBE_SERVER = 'sonarqube-server'   // Matches Manage Jenkins > System config
        // JFROG_REGISTRY = 'trialepv7i1.jfrog.io'
        // JFROG_REPO = 'petclinic-repo'
        // DOCKER_IMAGE = 'petclinic-app'
        // DOCKER_CONTAINER_PORT = '8082'

        SONARQUBE_SERVER = 'sonarqube-server'   // same as before
        JFROG_REGISTRY = 'triald122tk.jfrog.io' // new JFrog domain
        JFROG_REPO = 'petclinic-docker-local'   // repo key, not display name
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
                sh '''
                    mvn clean package -DskipTests
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh "mvn sonar:sonar -Dsonar.login=$SONAR_TOKEN"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "ls -lh target/"   // debug: confirm JAR exists
                sh "docker build -t ${DOCKER_IMAGE}:latest ."
            }
        }

        stage('Trivy Scan') {
            steps {
                sh "trivy image --exit-code 1 --severity HIGH,CRITICAL ${DOCKER_IMAGE}:latest"
            }
        }

        stage('Push to JFrog') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'jfrog-creds',
                                                     usernameVariable: 'JFROG_USER',
                                                     passwordVariable: 'JFROG_PASSWORD')]) {
                        sh """
                            echo "$JFROG_PASSWORD" | docker login ${JFROG_REGISTRY} -u "$JFROG_USER" --password-stdin
                            docker tag ${DOCKER_IMAGE}:latest ${JFROG_REGISTRY}/${JFROG_REPO}/${DOCKER_IMAGE}:latest
                            docker push ${JFROG_REGISTRY}/${JFROG_REPO}/${DOCKER_IMAGE}:latest
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
