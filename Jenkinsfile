
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
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Recuperation du projet depuis GitHub'
                checkout scm
            }
        }

        stage('Verification environnement') {
            steps {
                echo 'Verification de Java et Maven'
                sh 'java -version'
                sh 'mvn -version'
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
                    junit allowEmptyResults: true,
                          testResults: 'backend/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Generation du livrable') {
            steps {
                dir('backend') {
                    sh 'mvn -B package -DskipTests'
                }
            }
        }

        stage('Archivage du JAR') {
            steps {
                archiveArtifacts(
                    artifacts: 'backend/target/*.jar',
                    fingerprint: true
                )
            }
        }
    }

    post {
        success {
            echo 'BUILD SUCCESS : Livrable genere avec succes.'
        }

        failure {
            echo 'BUILD FAILURE : Consulter les logs.'
        }
    }
}

