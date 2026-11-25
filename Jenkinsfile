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

    environment {
        IMAGE = 'imencheikh/backend-spring'
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
                sh 'java -version'
                sh 'mvn -version'
                sh 'docker --version'
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

        stage('Docker Build') {
            steps {
                dir('backend') {
                    sh "docker build -t ${IMAGE}:${BUILD_NUMBER} -t ${IMAGE}:latest ."
                }
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_PASS')]) {
                    sh 'echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin'
                    sh "docker push ${IMAGE}:${BUILD_NUMBER}"
                    sh "docker push ${IMAGE}:latest"
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
        success {
            echo "BUILD SUCCESS : ${IMAGE}:${BUILD_NUMBER} publiee sur Docker Hub."
        }
        failure {
            echo 'BUILD FAILURE : Consulter les logs.'
        }
    }
}
