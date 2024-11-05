pipeline {
  agent any

  environment {
    SONARQUBE_URL = 'http://195.15.200.226:9000'
    SONAR_TOKEN = 'squ_24d84e105bcb5a38ea860b08ae3fe3b4b4ea4eef' // Nouveau token
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
        withSonarQubeEnv('token') { // Nom de l'installation de SonarQube
          sh '''
            mvn sonar:sonar \
              -Dsonar.projectKey=webapp \
              -Dsonar.host.url=$SONARQUBE_URL \
              -Dsonar.login=$SONAR_TOKEN \
              -Dsonar.ws.timeout=600 \
              -X
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
