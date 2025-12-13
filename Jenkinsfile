pipeline {
    agent { label 'maven-slave' }

    tools {
        maven 'Maven-3.9.11'
        jdk 'java21' // your configured JDK name
    }

    stages {
        stage('Build') {
            steps {
                sh '/opt/apache-maven-3.9.11/bin/mvn clean deploy'
            }
        }
    }
}
