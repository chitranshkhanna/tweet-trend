pipeline {
    agent { label 'maven-slave' }

    tools {
        maven 'Maven-3.9.11'
        jdk 'java8'
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean deploy'
            }
        }
    }
}
