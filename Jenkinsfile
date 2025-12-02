pipeline {
    agent {
        node {
            label 'maven-slave'
        }
    }

    stages {
        stage('Clone-code') {
            steps {
                git branch: 'https://github.com/chitranshkhanna/tweet-trend.git'
            }
        }
    }
}
