pipeline {
    agent { label 'maven-slave' }

    stages {
        stage('Clone Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/chitranshkhanna/tweet-trend.git'
            }
        }
    }
}
