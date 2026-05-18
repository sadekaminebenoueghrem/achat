pipeline {
    agent any

    tools {
        jdk 'jdk17'
    }

    environment {
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {

        stage('Code Checkout from GitHub') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/sadekaminebenoueghrem/achat.git'
            }
        }

        stage('Build Project') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                sh 'mvn org.owasp:dependency-check-maven:check'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                        mvn sonar:sonar -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Deploy to Nexus') {
            steps {
                sh 'mvn clean deploy -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t achat .
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                    docker run --rm \
                    -v /var/run/docker.sock:/var/run/docker.sock \
                    -v $HOME/.cache:/root/.cache \
                    aquasec/trivy image \
                    --scanners vuln \
                    --timeout 30m \
                    achat
                '''
            }
        }

        stage('Docker Compose Deployment') {
            steps {
                sh '''
                    docker-compose down -v || true
                    docker-compose up -d --build
                '''
            }
        }

        stage('OWASP ZAP Scan') {
            steps {
                sh '''
                    docker run --rm -t zaproxy/zap-stable \
                    zap-api-scan.py \
                    -t http://172.17.0.1:8089/SpringMVC/v3/api-docs \
                    -f openapi
                '''
            }
        }
    }
}
