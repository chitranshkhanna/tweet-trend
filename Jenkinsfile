pipeline {
    agent { label 'maven-slave' }

    tools {
        jdk 'java21'             // Java 21 configured in Jenkins
        maven 'Maven-3.9.11'     // Jenkins installs Maven automatically
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean deploy'
            }
        }
    }
}
