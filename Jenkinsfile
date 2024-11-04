pipeline {
  agent any
  
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

  stage('Build') {
    steps {
      sh 'mvn clean package'
    }
  }
}
