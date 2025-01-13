pipeline {
    agent any

    environment {
        awsCredentials = credentials('aws-credential1')
    }

    options {
        buildDiscarder(logRotator(daysToKeepStr: '30', numToKeepStr: '2'))
        timeout(time: 30, unit: 'MINUTES')
    }

    tools {
        maven 'Maven'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scmGit(
                    branches: [[name: 'main']], 
                    userRemoteConfigs: [[url: 'https://github.com/harshm1225/Super-Project.git']]
                )
            }
        }

        stage('Static Code Analysis') {
            steps {
                script {
                    def mvnPath = tool 'Maven'
                    withSonarQubeEnv('sonar-server') {
                        sh "${mvnPath}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=project1 -Dsonar.projectName='project1'"
                    }
                }
            }
        }

        stage('Build Package') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Upload to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: '3.85.87.111:8081',
                    groupId: 'addressbook',
                    version: '2.0-SNAPSHOT',
                    repository: 'maven-snapshots',
                    credentialsId: 'nexus-cred',
                    artifacts: [[
                        artifactId: 'project1',
                        file: 'target/addressbook-2.0.war',
                        type: 'war'
                    ]]
                )
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t 445617903595.dkr.ecr.us-east-1.amazonaws.com/demo1:${BUILD_NUMBER} .'
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 445617903595.dkr.ecr.us-east-1.amazonaws.com
                docker push 445617903595.dkr.ecr.us-east-1.amazonaws.com/demo1:${BUILD_NUMBER}
                docker tag 445617903595.dkr.ecr.us-east-1.amazonaws.com/demo1:${BUILD_NUMBER} 445617903595.dkr.ecr.us-east-1.amazonaws.com/demo1:latest
                docker push 445617903595.dkr.ecr.us-east-1.amazonaws.com/demo1:latest
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig(
                    credentialsId: 'kubernetes-token', 
                    serverUrl: 'https://4EAA92E5AE8FF95F0630C2B4DC736B33.gr7.us-east-1.eks.amazonaws.com'
                ) {
                    sh 'kubectl apply -f Application.yaml'
                }
            }
        }
    }
}
