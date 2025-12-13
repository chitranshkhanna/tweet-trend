pipeline {
    agent { label 'maven-slave' }

    tools {
        maven 'maven-3.9.11'
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
