pipeline {
    agent any

    tools {
        jdk 'jdk17'
    }
    
     environment {
    SONAR_TOKEN = credentials('sonar-token')
}

    stages {

        stage('Code checkout from GitHub') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/sadekaminebenoueghrem/achat.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

                stage('Build') {
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
      stage('Dockerize app') {
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

    stage('Dockerize project') {
          steps {
            sh '''
             docker build -t achat .
             docker-compose down -v || true
             docker-compose up -d --build
             '''
       }
    }
    stage('Dockerize project') {
          steps {
            sh '''
             docker run --rm -t zaproxy/zap-stable \
             zap-api-scan.py -t http://172.17.01.1:8089/SpringMVC/v3/api-docs -f openapi
             '''
       }
    }
   }
}
