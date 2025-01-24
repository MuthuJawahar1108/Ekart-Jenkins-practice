pipeline {
    agent any
    tools {
        jdk 'jdk11'
        maven 'maven3'
        dockerTool 'docker'
        
    }
    environment {
        SCANNER_HOME= tool 'sonar-scanner' 
    }

    stages {
        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }
        stage('COMPILE') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }
        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                sh ''' $SCANNER_HOME/bin/sonar-scanner \
                -Dsonar.java.binaries=. \
                -Dsonar.projectKey=Ekart-Muthu \
                -Dsonar.projectName=Ekart-Muthu \
                -Dsonar.login=$SONAR_TOKEN '''
            }
                }
            }
        }
        stage('BUILD') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        stage('docker build') {
            steps {
                      withDockerRegistry(credentialsId: 'docker-credentials', url: 'https://index.docker.io/v1/') {
                    sh "docker build -t e-cart:latest -f docker/Dockerfile ."
                    sh "docker tag e-cart muthujawahar/e-cart:latest"
                    sh "docker push muthujawahar/e-cart:latest"
                }
            }
        }
        stage('Deploy'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-credentials', url: 'https://index.docker.io/v1/') {
                        sh "docker run -d --name e-shop -p 8071:8071 muthujawahar/e-cart:latest "
                    }
                }
            }
        }

    }
}

