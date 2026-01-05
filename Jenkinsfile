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
                // Pulls the code from your Git repository
                checkout scm
            }
        }

        stage("Build and Package") {
            steps {
                // Compiles the Java code and creates the JAR file in the target folder
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
                // Separates the artifact for clean uploading
                sh "mkdir -p jarstaging && cp target/*.jar jarstaging/"
            }
        }

        stage("Jar Publish") {
            steps {
                script {
                    echo "<--------------- Jar Publish Started --------------->"
                    def server = Artifactory.newServer(url: registry + "/artifactory", credentialsId: "artifact-cred")
                    def props = "buildid=${env.BUILD_ID},commitid=${env.GIT_COMMIT}"
                    
                    // TARGET: Pointing directly to your physical 'libs-release-local' repository
                    // Using flat: true ensures the file is placed directly in the repo root
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "jarstaging/*.jar",
                                "target": "libs-release-local/",
                                "flat": "true",
                                "props": "${props}"
                            }
                        ]
                    }"""
                    
                    def buildInfo = server.upload(uploadSpec)
                    server.publishBuildInfo(buildInfo)
                    echo "<--------------- Jar Publish Ended --------------->"
                }
            }
        }

        stage("Archive") {
            steps {
                // Keeps a copy of the JAR within the Jenkins build history
                archiveArtifacts artifacts: "jarstaging/*.jar", fingerprint: true
            }
        }
    }
}