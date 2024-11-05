pipeline {
    agent any

    environment {
        // Définition des variables pour SonarQube et SSH
        SONARQUBE_URL = 'http://195.15.200.226:9000'
        SONAR_TOKEN = 'squ_24d84e105bcb5a38ea860b08ae3fe3b4b4ea4eef' // Nouveau token SonarQube
        SSH_AGENT_TOMCAT = '054d6250-dc24-4cb2-9fa3-baa3c332f955'    // ID SSH pour le serveur Tomcat (195.15.207.39)
        SSH_AGENT_SECURITY = 'security' // ID SSH pour le serveur Ubuntu où se trouvent JMeter et ZAP (188.213.130.79)
        TOMCAT_SERVER = '195.15.207.39'
        SECURITY_SERVER = '188.213.130.79'
        APP_NAME = 'webapp'
    }

    tools {
        // Utilisation de Maven pour la compilation
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
                          -Dsonar.projectKey=${APP_NAME} \
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

        stage('Deploy to Tomcat') {
            steps {
                sshagent(credentials: [SSH_AGENT_TOMCAT]) {
                    sh '''
                        scp target/${APP_NAME}.war debian@${TOMCAT_SERVER}:/opt/tomcat/webapps/
                    '''
                }
            }
        }

        stage('JMeter Performance Test') {
            steps {
                sshagent(credentials: [SSH_AGENT_SECURITY]) {
                    sh '''
                        ssh ubuntu@${SECURITY_SERVER} "jmeter -n -t /home/ubuntu/tests/${APP_NAME}_test_plan.jmx -l /home/ubuntu/rapport/jmeter_report.jtl -e -o /home/ubuntu/rapport/jmeter_report"
                    '''
                }
            }
        }

        stage('OWASP ZAP Security Test') {
            steps {
                sshagent(credentials: [SSH_AGENT_SECURITY]) {
                    sh '''
                        ssh ubuntu@${SECURITY_SERVER} "zap-cli quick-scan -r /home/ubuntu/rapport/zap_report.html http://${TOMCAT_SERVER}:8080/${APP_NAME}"
                    '''
                }
            }
        }
    }

    post {
        always {
            // Archive des rapports générés par JMeter et ZAP sur Jenkins
            archiveArtifacts artifacts: '**/rapport/*', allowEmptyArchive: true
            // Suppression des fichiers de rapport sur le serveur pour éviter l'accumulation
            sshagent(credentials: [SSH_AGENT_SECURITY]) {
                sh '''
                    ssh ubuntu@${SECURITY_SERVER} "rm -rf /home/ubuntu/rapport/*"
                '''
            }
        }
    }
}
