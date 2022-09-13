def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger'
]

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
            stage('Get commit details') {
                steps {
                    script {
                    env.GIT_COMMIT_MSG = sh (script: 'git log -1 --pretty=%B ${GIT_COMMIT}', returnStdout: true).trim()
                    env.GIT_AUTHOR = sh (script: 'git log -1 --pretty=%cn ${GIT_COMMIT}', returnStdout: true).trim()
                }
            }
        }
    }
    post{
        always{
            script{
                BUILD_USER = getBuildUser()
            }
            slackSend channel: '#test-slack',
                      color: COLOR_MAP[currentBuild.currentResult],
                      message: "*${currentBuild.currentResult}:* ${env.JOB_NAME} build ${env.BUILD_NUMBER} ${env.GIT_COMMIT_MSG} ${env.GIT_AUTHOR} by ${BUILD_USER} \n More information at: ${env.BUILD_URL}"
        }
    }
}
