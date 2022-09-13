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
    post {
        failure {
            slackSend (channel: '#test-slack', color: '#FF0000', message: """FAILED:
            Job: ${env.JOB_NAME}
            Build #${env.BUILD_NUMBER}
            Build: ${env.BUILD_URL})
            Comitted by: ${env.BUILD_USER}
            Last commit message: '${env.GIT_COMMIT_MSG}'""")
        }
        success {
            slackSend (channel: '#test-slack', color: '#00FF00', message: """SUCCESS:
            Job: ${env.JOB_NAME}
            Build #${env.BUILD_NUMBER}
            Build: ${env.BUILD_URL})
            Comitted by: ${env.BUILD_USER}
            Last commit message: '${env.GIT_COMMIT_MSG}'""")
        }
    }
}
