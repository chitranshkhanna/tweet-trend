pipeline {
    agent { label 'maven-slave' }

    tools {
        jdk 'java21'
        maven 'Maven-3.9.11'
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean deploy'  // uses tools automatically
            }
        }
    }
}
