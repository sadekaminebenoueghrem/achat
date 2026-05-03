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
                git branch: 'docker',
                    url: 'https://github.com/AladdinBelhaj/achat.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
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
      stage('Dockerize project') {
          steps {
            sh '''
             docker build -t achat .
             docker compose down -v || true
             docker compose up -d --build
             '''
       }
    }

   }
}
