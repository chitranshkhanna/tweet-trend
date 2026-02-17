def awsRegion = 'us-east-1'
def accountId = '374031960771'
def ecrRepo   = 'ttrend'
def imageName = "${accountId}.dkr.ecr.${awsRegion}.amazonaws.com/${ecrRepo}"
def version   = "${env.BUILD_NUMBER}"
def app

pipeline {
    agent { label "maven-slave" }

    tools {
        jdk "java17"
        maven "Maven-3.9.11"
    }

    options {
        timeout(time: 45, unit: 'MINUTES')
        disableConcurrentBuilds()
        timestamps()
        buildDiscarder(logRotator(
            numToKeepStr: '10',
            daysToKeepStr: '7',
            artifactNumToKeepStr: '5'
        ))
    }

    environment {
        MAVEN_OPTS = "-Xms512m -Xmx2g -XX:MaxMetaspaceSize=512m -XX:+UseG1GC"
        JAVA_TOOL_OPTIONS = "-Xms512m -Xmx2g -XX:MaxMetaspaceSize=512m -XX:+UseG1GC"
    }

    stages {

        stage("Pre-clean Workspace") {
            steps {
                cleanWs(deleteDirs: true)
            }
        }

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

        stage("Quality Gate") {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Verify AWS Access") {
            steps {
                withAWS(credentials: 'aws-jenkins', region: awsRegion) {
                    sh 'aws sts get-caller-identity'
                }
            }
        }

        // ✅ FIXED CODEARTIFACT STAGE
        stage("Jar Publish to CodeArtifact") {
            steps {
                withAWS(credentials: 'aws-jenkins', region: awsRegion) {
                    sh """
                        export CODEARTIFACT_AUTH_TOKEN=\$(aws codeartifact get-authorization-token \
                          --domain my-domain \
                          --domain-owner ${accountId} \
                          --query authorizationToken \
                          --output text)

                        mvn deploy -DskipTests
                    """
                }
            }
        }

        stage("Docker Build") {
            steps {
                script {
                    app = docker.build("${imageName}:${version}")
                }
            }
        }

        stage("Login to ECR") {
            steps {
                withAWS(credentials: 'aws-jenkins', region: awsRegion) {
                    sh """
                      aws ecr get-login-password \
                      | docker login \
                        --username AWS \
                        --password-stdin ${accountId}.dkr.ecr.${awsRegion}.amazonaws.com
                    """
                }
            }
        }

        stage("Docker Push") {
            steps {
                script {
                    app.push()
                }
            }
        }
    }

    post {

        success {
            echo "✅ Build succeeded"
        }

        failure {
            echo "❌ Build failed — check logs"
        }

        always {
            echo "🧹 Cleaning workspace & Docker"

            cleanWs(deleteDirs: true)

            // safer cleanup (only remove this image)
            sh "docker rmi ${imageName}:${version} || true"
        }
    }
}
