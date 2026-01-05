pipeline {
    agent { label "maven-slave" }

    tools {
        jdk "java17"
        maven "Maven-3.9.11"
    }

    stages {
        stage("Checkout") {
            steps {
                checkout scm
            }
        }

        stage("Build and Package") {
            steps {
                sh """
                    echo "JAVA_HOME=$JAVA_HOME"
                    java -version
                    mvn clean package -DskipTests
                """
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    def scannerHome = tool(
                        name: "valaxy-sonar-scanner",
                        type: "hudson.plugins.sonar.SonarRunnerInstallation"
                    )

                    withSonarQubeEnv("valaxy-sonarqube-server") {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=demo-workshop \
                            -Dsonar.projectName=demo-workshop \
                            -Dsonar.sources=src/main/java \
                            -Dsonar.java.binaries=target/classes
                        """
                    }
                }
            }
        }

        stage("Create jarstaging in Workspace") {
            steps {
                sh """
                    mkdir -p jarstaging
                    cp target/*.jar jarstaging/
                """
            }
        }

        stage("Archive Artifact") {
            steps {
                archiveArtifacts artifacts: "jarstaging/*.jar", fingerprint: true
            }
        }
    }
}