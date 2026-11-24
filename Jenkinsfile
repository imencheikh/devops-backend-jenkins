pipeline {
    agent any

    tools {
        jdk 'JDK'
        maven 'Maven'
    }

    triggers {
        pollSCM('* * * * *')
    }

    options {
        skipDefaultCheckout(true)
    }

    stages {
        stage('Recuperer le projet') {
            steps {
                checkout scm
            }
        }

        stage('Tests unitaires') {
            steps {
                dir('backend') {
                    sh 'mvn -B clean test'
                }
            }
            post {
                always {
                    junit 'backend/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Creer le livrable') {
            steps {
                dir('backend') {
                    sh 'mvn -B package -DskipTests'
                }
                archiveArtifacts artifacts: 'backend/target/*.jar', fingerprint: true
            }
        }
    }
}
