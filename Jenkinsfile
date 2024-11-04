pipeline {
  agent any
  
  environment {
    SONAR_TOKEN = credentials('sonar_token') // Jeton pour SonarQube
    SONARQUBE_URL = 'http://195.15.200.226:9000' // URL de SonarQube
    PRODUCTION_SERVER = '195.15.207.39' // IP du serveur Tomcat
    SECURITY_SERVER = '188.213.128.116' // IP du serveur OWASP ZAP
  }
  
  tools {
    maven 'Apache Maven 3.8.7' // Maven installé dans Jenkins
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

    stage('SAST - SonarQube Analysis') {
      steps {
        sshagent(['sonarqube']) { // Utiliser les credentials pour SonarQube
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

    stage('Deploy to Tomcat') {
      steps {
        sshagent(['054d6250-dc24-4cb2-9fa3-baa3c332f955']) { // Clé SSH pour le serveur de production (Tomcat)
          sh 'scp -o StrictHostKeyChecking=no target/*.war debian@$PRODUCTION_SERVER:/opt/tomcat/webapps/webapp.war'
        }
      }
    }

    stage('DAST - OWASP ZAP Analysis') {
      steps {
        sshagent(['security']) { // Clé SSH pour le serveur de sécurité (OWASP ZAP)
          sh '''
            ssh -o StrictHostKeyChecking=no debian@$SECURITY_SERVER \
              "zap.sh -daemon -host $PRODUCTION_SERVER -port 8080 -t http://$PRODUCTION_SERVER:8080/webapp -r zap_report.html"
          '''
        }
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'zap_report.html', allowEmptyArchive: true
      echo "Build ${currentBuild.fullDisplayName} completed with ${currentBuild.currentResult}."
      cleanWs()
    }
  }
}
