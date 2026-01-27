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

        /* ================= AWS VERIFICATION (SAFE) ================= */
        stage('Verify AWS Access') {
            steps {
                withAWS(credentials: 'aws-jenkins', region: awsRegion) {
                    sh 'aws sts get-caller-identity'
                    sh 'aws codeartifact list-repositories'
                    sh 'aws ecr describe-repositories'
                }
            }
        }

        /* ================= JAR PUBLISH (AWS CodeArtifact) ================= */
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

        /* ================= DOCKER BUILD ================= */
        stage("Docker Build") {
            steps {
                script {
                    echo '<--------------- Docker Build Started --------------->'
                    app = docker.build("${imageName}:${version}")
                    echo '<--------------- Docker Build Ends --------------->'
                }
            }
        }

        /* ================= ECR LOGIN ================= */
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

        /* ================= DOCKER PUSH ================= */
        stage("Docker Publish") {
            steps {
                script {
                    echo '<--------------- Docker Publish Started --------------->'
                    app.push()
                    echo '<--------------- Docker Publish Ended --------------->'
                }
            }
        }
    }
}
