pipeline {
    agent any 
    tools {
        maven "MAVEN 3.9.9"
        jdk "JAVA 17"
    }

    stages{
        stage('Fetch code') {
            steps {
                git branch: 'atom', url: 'https://github.com/hkhcoder/vprofile-project.git'
            }           
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage ('Deploy') {
            steps{
                sh 'mvn install -DskipTests'
            }
            post {
                success{
                    echo "Archiving the artifact"
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
    }
}