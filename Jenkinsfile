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

        stage('Build & Test') {
            steps {
                sh '''
                    echo "JAVA_HOME=$JAVA_HOME"
                    javac -version
                    mvn clean test
                '''
            }
        }

        stage('SonarQube analysis') {
            environment {
                SCANNER_HOME = tool 'valaxy-sonar-scanner'
            }
            steps {
                withSonarQubeEnv('valaxy-sonarqube-server') {
                    sh """
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=demo-workshop \
                        -Dsonar.projectName=demo-workshop \
                        -Dsonar.sources=src/main/java \
                        -Dsonar.tests=src/test/java \
                        -Dsonar.java.binaries=target/classes
                    """
                }
            }
        }
    }
}
