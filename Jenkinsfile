pipeline {
  agent any
  
  environment {
    SONAR_TOKEN = credentials('sonarqube') // Token for SonarQube
    SONARQUBE_URL = 'http://195.15.200.226:9000' // SonarQube server URL
    PRODUCTION_SERVER = '195.15.207.39' // Tomcat server IP
    SECURITY_SERVER = '188.213.128.116' // OWASP ZAP server IP
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
    
    stage('Check-Git-Secrets') {
      steps {
        sh 'rm -f trufflehog || true'
        sh 'docker run gesellix/trufflehog --json https://github.com/Trohsamuel/webapp.git > trufflehog'
        sh 'cat trufflehog'
      }
    }
    
    stage('Source Composition Analysis') {
      steps {
        sh 'rm -f owasp* || true'
        sh 'wget "https://raw.githubusercontent.com/Trohsamuel/webapp/master/owasp-dependency-check.sh"'
        sh 'chmod +x owasp-dependency-check.sh'
        sh 'bash owasp-dependency-check.sh'
        sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
      }
    }
    
    stage('SAST') {
      steps {
        withSonarQubeEnv('sonarqube') {
          sh '''
            mvn sonar:sonar \
              -Dsonar.projectKey=webapp \
              -Dsonar.host.url=$SONARQUBE_URL \
              -Dsonar.login=$SONAR_TOKEN
          '''
          sh 'cat target/sonar/report-task.txt'
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
        sshagent(['054d6250-dc24-4cb2-9fa3-baa3c332f955']) {
          sh 'scp -o StrictHostKeyChecking=no target/*.war debian@195.15.207.39:/prod/apache-tomcat-9.0.96/webapps/webapp.war'
        }
      }
    }
    
    stage('DAST') {
      steps {
        sshagent(['security']) {
          sh 'ssh -o StrictHostKeyChecking=no debian@188.213.128.116 "zap.sh -daemon -quickurl http://195.15.207.39:8080/webapp/ -quickout zap_report.html" || true'
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
