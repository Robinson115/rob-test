def getBuildUser(){
    return currentBuild.rawBuild.getCause(Cause.UserIdCause).getUserId()
}

var name={'aravind'}


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
    post {
        always{
            script {
                if (env.BRANCH_NAME == 'master')
                env.GIT_COMMIT_CHANGES = sh (script: 'git log -1 --pretty=%B ${GIT_COMMIT}', returnStdout: true).trim()
                env.GIT_COMMIT_MSG = sh (script: 'git log -1 --pretty=%B ${GIT_COMMIT}', returnStdout: true).trim()
                env.GIT_AUTHOR = sh (script: 'git log -1 --pretty=%cn ${GIT_COMMIT}', returnStdout: true).trim()
                BUILD_USER = getBuildUser()  
            } 
            
        }                   
        failure {
            slackSend (channel: '#test-slack', color: '#FF0000', message: """FAILED:
            Job: ${env.JOB_NAME}
            Build: #${env.BUILD_NUMBER}
            Build Done by: ${BUILD_USER}
            Build: ${env.BUILD_URL})
            Comitted by: ${env.GIT_AUTHOR}
            Last commit message: '${env.GIT_COMMIT_MSG}'""")
        }
        success {
            slackSend (channel: '#test-slack', color: '#00FF00', message: """SUCCESS:
            Job: ${env.JOB_NAME}
            Build: #${env.BUILD_NUMBER}
            Build Done by: ${BUILD_USER}
            Build: ${env.BUILD_URL})
            Comitted by: ${env.GIT_AUTHOR}
            Last commit message: '${env.GIT_COMMIT_MSG}'""")
        }
    }
}
