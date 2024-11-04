pipeline {
  agent any
  
  environment {
    SONAR_TOKEN = credentials('sonar_token') // Utilisez 'sonar_token' comme nom du credential pour SonarQube
    SONARQUBE_URL = 'http://195.15.200.226:9000' // URL du serveur SonarQube
    PRODUCTION_SERVER = '195.15.207.39' // IP du serveur Tomcat
    SECURITY_SERVER = '188.213.128.116' // IP du serveur OWASP ZAP 
  }
  
  tools {
    maven 'Apache Maven 3.8.7'
  }
  
  stages {
    stage('SAST') {
      steps {
        withSonarQubeEnv('sonar_token') { // Utilisez 'sonar_token' pour se connecter à SonarQube
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
    
    stage('Deploy-To-Tomcat') {
      steps {
        sshagent(['054d6250-dc24-4cb2-9fa3-baa3c332f955']) { // Identifiant Jenkins pour la connexion au serveur Tomcat
          sh 'scp -o StrictHostKeyChecking=no target/*.war debian@$PRODUCTION_SERVER:/opt/tomcat/webapps/webapp.war'
        }
      }
    }
    
    stage('DAST') {
      steps {
        sshagent(['security']) { // Identifiant Jenkins pour la connexion au serveur ZAP
          sh '''
            ssh -o StrictHostKeyChecking=no debian@$SECURITY_SERVER \
              "zap.sh -daemon -port 8080 -host $PRODUCTION_SERVER -config api.disablekey=true \
              && zap-baseline.py -t http://$PRODUCTION_SERVER:8080/webapp/ -r zap_report.html"
          '''
        }
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'zap_report.html', allowEmptyArchive: true
      echo "Build ${currentBuild.fullDisplayName} has ${currentBuild.currentResult}."
      cleanWs()
    }
  }
}
