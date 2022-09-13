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
    }
    post{
        always{
            script{
                BUILD_USER = getBuildUser()
            }
            slackSend channel: '#test-slack',
                      color: COLOR_MAP[currentBuild.currentResult],
                      message: "*${currentBuild.currentResult}:* ${env.JOB_NAME} build ${env.BUILD_NUMBER} by ${BUILD_USER} \n More information at: ${env.CHANGE_TITLE} ${env.CHANGE_AUTHOR} ${env.BUILD_URL}"
        }
    }
}
