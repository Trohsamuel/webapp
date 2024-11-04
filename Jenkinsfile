pipeline {
  agent any
  
  tools {
    maven 'Apache Maven 3.8.7'
  }
  
  stages {
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
