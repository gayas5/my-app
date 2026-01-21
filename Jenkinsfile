pipeline {
    agent any

    tools {
        maven 'maven'
        jdk 'JDK21'
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
