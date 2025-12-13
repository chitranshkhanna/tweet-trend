pipeline {
    agent { label 'maven-slave' }

    tools {
        jdk 'java21'
        maven 'Maven-3.9.11'
    }

    environment {
        JAVA_HOME = tool name: 'java21', type: 'jdk'
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean deploy'
            }
        }
    }
}
