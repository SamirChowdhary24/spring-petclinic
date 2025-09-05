pipeline {
    agent any

    tools {
        maven 'Maven-3.8.7'
        jdk 'java-17-openjdk'  // name must match what you added in Jenkins tool config
    }

    environment {
        SONARQUBE = 'sonarqube-server'   // name you set in Jenkins
        SONAR_TOKEN = credentials('sonar-token')
        JFROG_CREDS = credentials('jfrog-creds')
        DOCKER_REPO = "trialepv7i1.jfrog.io/petclinic-docker"  // replace with your repo name
        IMAGE_NAME = "petclinic-app"
        IMAGE_TAG = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/YOUR-REPO/petclinic.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh "mvn sonar:sonar -Dsonar.login=${SONAR_TOKEN}"
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                sh """
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                trivy image --exit-code 1 --severity HIGH,CRITICAL ${IMAGE_NAME}:${IMAGE_TAG} || true
                """
            }
        }

        stage('Push to JFrog') {
            steps {
                sh """
                echo ${JFROG_CREDS_PSW} | docker login -u ${JFROG_CREDS_USR} trialepv7i1.jfrog.io --password-stdin
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG}
                docker push ${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy Locally') {
            steps {
                sh """
                docker rm -f petclinic || true
                docker run -d --name petclinic -p 8082:8080 ${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
    }
}
