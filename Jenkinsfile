pipeline {
    agent any

    tools {
        maven 'maven-3'
        jdk 'JDK 21'
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn -version'
                sh 'java -version'
                sh 'mvn clean package'
            }
        }
    }
}

