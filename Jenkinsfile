def registry = "https://trialolvllu.jfrog.io"

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
                sh 'mvn clean package -DskipTests'
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    def scannerHome = tool(name: "valaxy-sonar-scanner", type: "hudson.plugins.sonar.SonarRunnerInstallation")
                    withSonarQubeEnv("valaxy-sonarqube-server") {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=demo-workshop -Dsonar.projectName=demo-workshop -Dsonar.sources=src/main/java -Dsonar.java.binaries=target/classes"
                    }
                }
            }
        }

        stage("Create jarstaging") {
            steps {
                sh "mkdir -p jarstaging && cp target/*.jar jarstaging/"
            }
        }

        stage("Jar Publish") {
            steps {
                script {
                    def server = Artifactory.newServer(url: registry + "/artifactory", credentialsId: "artfiact-cred")
                    def props = "buildid=${env.BUILD_ID},commitid=${env.GIT_COMMIT}"
                    // Using a single-line string to prevent bracket mismatch errors
                    def uploadSpec = """{"files": [{"pattern": "jarstaging/(*)","target": "libs-release-local/{1}","flat": "false","props": "${props}","exclusions": ["*.sha1", "*.md5"]}]}"""
                    server.upload(uploadSpec)
                }
            }
        }

        stage("Archive") {
            steps {
                archiveArtifacts artifacts: "jarstaging/*.jar", fingerprint: true
            }
        }
    }
}