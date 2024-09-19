@Library('jenkins-library@main') _

pipeline {
    agent any
    tools {
        nodejs "node"
    }
    stages {
        stage('increment version') {
            steps {
                script {
                    incrementVersion()
                }
            }
        }
        
        stage('run tests') {
            steps {
                script {
                    runTests()
                }
            }
        }
        
        stage('Build and Push docker image') {
            steps {
                script {
                    dockerBuild()
                }
            }
        }

        stage('deploy to EC2') {
            steps {
                script {
                   def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME}"
                   def ec2Instance = "ec2-user@18.206.158.122"

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
                    gitCommit()
                }
            }
        }
    }
}