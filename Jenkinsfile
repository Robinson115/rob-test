def getBuildUser(){
    return currentBuild.rawBuild.getCause(Cause.UserIdCause).getUserId()
}

pipeline {
    agent any
    
    environment{
        BUILD_USER=''
    }
    stages {
        stage('build') {
            steps {
                echo "Hello World!"
            }
        }
    }    
    stages{
         stage('Get commit details') {
            steps {
                script {
                    env.GIT_COMMIT_MSG = sh (script: 'git log -1 --pretty=%B ${GIT_COMMIT}', returnStdout: true).trim()
                    env.GIT_AUTHOR = sh (script: 'git log -1 --pretty=%cn ${GIT_COMMIT}', returnStdout: true).trim()
                    BUILD_USER = getBuildUser()  
                }
            }
        }        
    }    

    post {
        failure {
            slackSend (channel: '#test-slack', color: '#FF0000', message: """FAILED:
            y:${BUILD_USER}
            Job: ${env.JOB_NAME}
            Build: #${env.BUILD_NUMBER}
            Build: ${env.BUILD_URL})
            Comitted by: ${env.GIT_AUTHOR}
            Last commit message: '${env.GIT_COMMIT_MSG}'""")
        }
        success {
            slackSend (channel: '#test-slack', color: '#00FF00', message: """SUCCESS:
            By:${BUILD_USER}
            Job: ${env.JOB_NAME}
            Build: #${env.BUILD_NUMBER}
            Build: ${env.BUILD_URL})
            Comitted by: ${env.GIT_AUTHOR}
            Last commit message: '${env.GIT_COMMIT_MSG}'""")
        }
    }
}
