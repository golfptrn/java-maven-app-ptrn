#!/usr/bin/env groovy

library indentifier: 'jenkins-shared-library@main', retriever: modernSCM(
    [$class: 'GitSCMSource', 
    remote: 'https://github.com/golfptrn/jenkins-shared-library.git',
    credentialsId: 'GitHub-credential']

def gv

pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    stages {
        stage("init") {
            steps {
                script {
                    gv = load "script.groovy"
                }
            }
        }
        stage("build jar") {
            steps {
                script {
                    buildJar()
                }
            }
        }
        stage("build and push image") {
            steps {
                script {
                    buildImage 'golfpongtarin/demo-app:1.1'
                    dockerLogin()
                    dockerPush 'golfpongtarin/demo-app:1.1'
                }
            }
        }
        stage("deploy") {
            steps {
                script {
                    gv.deployApp()
                }
            }
        }
    }   
}
