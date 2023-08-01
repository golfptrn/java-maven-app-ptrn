def gv

pipeline {
    agent any
    parameters {
        string (name: 'VERSION', defualtValue: '', description: 'version to deploy on prod')
        choice (name: 'VERSION', choices: ['1.1.0', '1.2.0', '1.3.0']. description: '')
        booleanParam (name: 'executeTests', defaultValue: true, description: '')
    }
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
                    gv.buildJar()
                }
            }
        }
        stage("build image") {
            steps {
                script {
                    gv.buildImage()
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
