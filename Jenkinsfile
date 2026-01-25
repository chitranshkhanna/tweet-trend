def registry = 'https://trialolvllu.jfrog.io'
def imageName = 'trialolvllu.jfrog.io/valaxy-docker-local/ttrend'
def version = '2.1.4'
def app   // <-- global scope for docker image

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

        stage("Create jarstaging") {
            steps {
                sh "mkdir -p jarstaging && cp target/*.jar jarstaging/"
            }
        }

        stage('Verify AWS Access') {
    steps {
        withAWS(credentials: 'aws-jenkins', region: 'us-east-1') {
            sh 'aws sts get-caller-identity'
            sh 'aws codeartifact list-repositories --domain my-domain'
            sh 'aws ecr describe-repositories'
        }
    }
}


        stage("Jar Publish") {
            steps {
                script {
                    echo "<--------------- Jar Publish Started --------------->"

                    def server = Artifactory.newServer(
                        url: registry + "/artifactory",
                        credentialsId: "artifact-cred"
                    )

                    def props = "buildid=${env.BUILD_ID},commitid=${env.GIT_COMMIT}"

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
                archiveArtifacts artifacts: "jarstaging/*.jar", fingerprint: true
            }
        }

        stage("Docker Build") {
            steps {
                script {
                    echo '<--------------- Docker Build Started --------------->'
                    app = docker.build("${imageName}:${version}")
                    echo '<--------------- Docker Build Ends --------------->'
                }
            }
        }

        stage("Docker Publish") {
            steps {
                script {
                    echo '<--------------- Docker Publish Started --------------->'
                    docker.withRegistry(registry, 'artifact-cred') {
                        app.push()
                    }
                    echo '<--------------- Docker Publish Ended --------------->'
                }
            }
        }

    }
}
