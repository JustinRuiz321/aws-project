@Library('jenkins-library@main') _

pipeline {
    agent any
    tools {
        nodejs "node"
    }
    stages {
        stage('increment version') {
            when {
                expression {
                    return env.GIT_BRANCH == "master"
            }
            steps {
                script {
                    incrementVersion()
                }
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
            when {
                expression {
                    return env.GIT_BRANCH == "master"
                }
            }

            steps {
                script {
                    dockerBuild()
                }
            }
        }

        stage('deploy to EC2') {
            when {
                expression {
                    return env.GIT_BRANCH == "master"
                }
            }

            steps {
                script {
                   def shellCmd = "bash ./serverCmds.sh ${IMAGE_NAME}"
                   def ec2Instance = "ec2-user@18.206.158.122"

                   sshagent(['ec2-keypair']) {
                       sh "scp -o StrictHostKeyChecking=no serverCmds.sh ${ec2Instance}:/home/ec2-user"
                       sh "scp -o StrictHostKeyChecking=no docker-compose.yaml ${ec2Instance}:/home/ec2-user"
                       sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
                   }
                }
            }
        }
        
        stage('commit version update') {
            when {
                expression {
                    return env.GIT_BRANCH == "master"
                }
            }

            steps {
                script {
                        withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                            sh 'git config --global user.email "jenkins@me.com"'
                            sh 'git config --global user.name "jenkins"'
                            sh "git remote set-url origin https://$USER:$PASS@github.com/JustinRuiz321/aws-project.git"
                            sh 'git add .'
                            sh 'git commit -m "ci: version bump"'
                            sh 'git push origin HEAD:main'
                }
            }
        }
    }
}
}