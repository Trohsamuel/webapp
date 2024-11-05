pipeline {
  agent any

  environment {
    SONARQUBE_URL = 'http://195.15.200.226:9000'
  }

  tools {
    maven 'Apache Maven 3.8.7'
  }

  stages {
    stage('Initialize') {
      steps {
        sh '''
          echo "PATH = ${PATH}"
          echo "M2_HOME = ${M2_HOME}"
        '''
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('sonar_token') {
          sh '''
            mvn sonar:sonar \
              -Dsonar.projectKey=webapp \
              -Dsonar.host.url=$SONARQUBE_URL \
              -Dsonar.login=$SONAR_TOKEN
          '''
        }
      }
    }

    stage('Build') {
      steps {
        sh 'mvn clean package'
      }
    }
  }

  post {
    always {
      cleanWs() // Nettoie l'espace de travail après chaque build
      echo "Build ${currentBuild.fullDisplayName} has ${currentBuild.currentResult}."
    }
  }
}
