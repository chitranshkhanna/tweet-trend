pipeline {
    agent { label 'maven-slave' }

    tools {
        jdk 'java21'            // Name of JDK configured in Jenkins
        maven 'Maven-3.9.11'    // Name of Maven configured in Jenkins
    }

    environment {
        // Automatically resolve JDK and Maven paths
        JAVA_HOME = tool(name: 'java21', type: 'jdk')
        MAVEN_HOME = tool(name: 'Maven-3.9.11', type: 'maven')
        PATH = "${JAVA_HOME}/bin:${MAVEN_HOME}/bin:${env.PATH}"
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
