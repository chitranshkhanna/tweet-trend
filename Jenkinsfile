pipeline {
    agent { label 'maven-slave' }

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-21'  // path to Java 21 on agent
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
    }

    stages {
        stage('Build') {
            steps {
                sh '/opt/apache-maven-3.9.11/bin/mvn clean deploy'
            }
        }
    }
}
