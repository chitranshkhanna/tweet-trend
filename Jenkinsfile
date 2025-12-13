pipeline {
    agent { label 'maven-slave' }

    tools {
        jdk 'java21'           // Must match Jenkins JDK name
        maven 'Maven-3.9.11'   // Must match Jenkins Maven name
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                // Force JAVA_HOME for this shell session
                sh '''
                    export JAVA_HOME=$(tool name: "java21" type: "jdk")
                    export PATH=$JAVA_HOME/bin:$PATH
                    mvn clean deploy
                '''
            }
        }
    }
}
