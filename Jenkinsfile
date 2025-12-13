pipeline {
    agent { label 'maven-slave' }

    stages {
        stage('Build') {
            steps {
                sh '/opt/apache-maven-3.9.11/bin/mvn clean deploy'
            }
        }
    }
}
