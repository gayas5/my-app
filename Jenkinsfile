pipeline {
    agent any

    tools {
        maven 'maven-3'
        jdk 'java-21'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/<your-username>/myapp.git'
            }
        }

        stage('Build') {
            steps {
                dir('my-app') {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir('my-app') {
                    sh 'docker build -t myapp:latest .'
                }
            }
        }
    }
}
