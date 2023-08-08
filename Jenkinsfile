#!/usr/bin/env groovy

library identifier: 'jenkins-shared-library@main', retriever: modernSCM(
    [$class: 'GitSCMSource', 
    remote: 'https://github.com/golfptrn/jenkins-shared-library.git',
    credentialsId: 'GitHub-credential'
    ]
)

def gv

pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        IMAGE_NAME = 'golfpongtarin/demo-app:java-maven-1.0'
    }
    stages {
        stage("build jar") {
            steps {
                script {
                    echo 'Building the application...'
                    buildJar()
                }
            }
        }
        stage("build and push image") {
            steps {
                script {
                    echo 'Building docker image..'
                    buildImage (env.IMAGE_NAME)
                    dockerLogin()
                    dockerPush (env.IMAGE_NAME)
                }
            }
        }
        stage("deploy") {
            steps {
                script {
                    echo 'deploying docker image to EC2...'
                    def dockerCmd = "docker run -p 8080:8080 -d ${IMAGE_NAME}"
                    sshagent(['ec2-server-key']) {
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@13.40.100.59 ${dockerCmd}"
                    }
                }
            }
        }
    }   
}
