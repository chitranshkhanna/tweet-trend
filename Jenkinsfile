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
    }
}
