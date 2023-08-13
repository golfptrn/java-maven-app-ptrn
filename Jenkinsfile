#!/usr/bin/env groovy

library identifier: 'jenkins-shared-library@main', retriever: modernSCM(
    [$class: 'GitSCMSource',
    remote: 'https://github.com/golfptrn/jenkins-shared-library.git',
    credentialsId: 'GitHub-credential'
    ]
)

pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    stages {
        stage("increment version") {
            script {
                echo 'incrementing app version...'
                sh "mvn build-helper:parse-version versions:set \
                   -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                   versions:commit"
                def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                def version = matcher[0][1]
                env.IMAGE_NAME = "$version-$BUILD_NUMBER"
            }
        }
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
                    def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME}"
                    def ec2Instance = "ec2-user@3.11.70.125"
                    sshagent(['ec2-server-key']) {
                        sh "scp -o StrictHostKeyChecking=no server-cmds.sh ${ec2Instance}:/home/ec2-user"
                        sh "scp -o StrictHostKeyChecking=no docker-compose.yaml ${ec2Instance}:/home/ec2-user"
                        sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
                    }
                }
            }
        }
        stage('commit version update') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-repo', passwordVariable: 'PASS', usernameVariable: 'USER']) {
                        sh "git remote set-url origin https//${USER}:${PASS}@github.com/golfptrn/java-maven-app-ptrn.git"
                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        sh 'git push origin HEAD:jenkins-jobs'
                    }
                }
            }
        }
    }
}
