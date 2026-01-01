pipeline {
    agent { label 'maven-slave' }

    tools {
        jdk 'java17'
        maven 'Maven-3.9.11'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh '''
                  echo "JAVA_HOME=$JAVA_HOME"
                  javac -version
                  mvn clean package
                '''
            }
        }
        stage('SonarQube analysis') {
        environment {
          scannerHome = tool 'valaxy-sonar-scanner'
        }
        steps{
        withSonarQubeEnv('valaxy-sonarqube-server') { // If you have configured more than one global server connection, you can specify its name
          sh "${scannerHome}/bin/sonar-scanner"
    }
    }
  }
    }
}
