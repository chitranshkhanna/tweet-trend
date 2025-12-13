pipeline {
    agent { label 'maven-slave' }

    tools {
        jdk 'java21'           // Jenkins configured JDK name
        maven 'Maven-3.9.11'   // Jenkins configured Maven name
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    // Resolve the JDK path using the Jenkins tool step
                    def javaHome = tool name: 'java21', type: 'jdk'
                    // Resolve Maven path (optional, if not using PATH)
                    def mavenHome = tool name: 'Maven-3.9.11', type: 'maven'

                    // Use a shell block with proper environment
                    sh """
                        export JAVA_HOME=${javaHome}
                        export PATH=\$JAVA_HOME/bin:\$PATH
                        mvn clean deploy
                    """
                }
            }
        }
    }
}
