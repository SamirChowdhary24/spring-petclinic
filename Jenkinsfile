pipeline {
    agent any

    tools {
        maven 'Maven3'
        jdk 'jdk17'
    }

    environment {
        DOCKER_IMAGE = "petclinic-app"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/spring-projects/spring-petclinic.git'
            }
        }

        stage('Build & Test (MySQL only)') {
            steps {
                sh '''
                    echo "⚡ Running only MySQL integration tests..."
                    mvn clean test -Dtest=*MySqlIntegrationTests
                '''
            }
        }

        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool 'SonarQubeScanner'
            }
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "📂 Checking target directory before Docker build..."
                    ls -lh target/ || echo "❌ No target folder found!"
                    
                    echo "🐳 Building Docker image..."
                    docker build -t ${DOCKER_IMAGE}:latest .
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                    echo "🔍 Running Trivy Scan..."
                    trivy image --exit-code 0 --severity LOW,MEDIUM,HIGH ${DOCKER_IMAGE}:latest
                '''
            }
        }

        stage('Push to JFrog') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jfrog-creds', passwordVariable: 'JFROG_PASSWORD', usernameVariable: 'JFROG_USERNAME')]) {
                    sh '''
                        echo "📦 Logging into JFrog..."
                        echo "$JFROG_PASSWORD" | docker login myjfrogrepo.jfrog.io -u "$JFROG_USERNAME" --password-stdin

                        echo "📤 Pushing Docker image..."
                        docker tag ${DOCKER_IMAGE}:latest myjfrogrepo.jfrog.io/${DOCKER_IMAGE}:latest
                        docker push myjfrogrepo.jfrog.io/${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    echo "🚀 Deploying container locally..."
                    docker run -d -p 8082:8080 --name petclinic ${DOCKER_IMAGE}:latest
                '''
            }
        }
    }
}
