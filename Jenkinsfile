pipeline {
    agent { label 'maven-slave' }

    environment {
        PATH = "/opt/apache-maven-3.9.11/bin:${env.PATH}"
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean deploy'
            }
        }
    }
}
