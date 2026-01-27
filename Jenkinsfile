def awsRegion = 'us-east-1'
def accountId = '374031960771'
def ecrRepo   = 'ttrend'
def imageName = "${accountId}.dkr.ecr.${awsRegion}.amazonaws.com/${ecrRepo}"
def version   = '2.1.4'
def app

pipeline {
    agent { label "maven-slave" }

    tools {
        jdk "java17"
        maven "Maven-3.9.11"
    }

    /* ================= PIPELINE SAFETY ================= */
    options {
        timeout(time: 45, unit: 'MINUTES')          // kill hung builds
        disableConcurrentBuilds()                    // no parallel memory fights
        timestamps()                                 // readable logs
        buildDiscarder(logRotator(
            numToKeepStr: '10',                      // keep last 10 builds
            daysToKeepStr: '7',                      // or max 7 days
            artifactNumToKeepStr: '5'
        ))
    }

    /* ================= MEMORY SETTINGS ================= */
    environment {
        MAVEN_OPTS = "-Xms512m -Xmx2g -XX:MaxMetaspaceSize=512m -XX:+UseG1GC"
        JAVA_TOOL_OPTIONS = "-Xms512m -Xmx2g -XX:MaxMetaspaceSize=512m -XX:+UseG1GC"
    }

    stages {

        /* ================= PRE-CLEAN ================= */
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
                    try {
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
                            -Dsonar.java.binaries=target/classes \
                            -Dsonar.ce.javaOpts="-Xmx1g"
                            """
                        }
                    } catch (err) {
                        echo "⚠️ Sonar failed, continuing build"
                    }
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

        stage("Jar Publish to CodeArtifact") {
            steps {
                withAWS(credentials: 'aws-jenkins', region: awsRegion) {
                    sh """
                      aws codeartifact login \
                        --tool maven \
                        --domain my-domain \
                        --domain-owner ${accountId} \
                        --repository my-maven-repo \
                        --region ${awsRegion}

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

    /* ================= ALWAYS EXECUTED ================= */
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

            sh '''
              docker system prune -af || true
              docker volume prune -f || true
            '''
        }
    }
}
