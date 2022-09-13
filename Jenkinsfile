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
            success {
                slackSend channel: "#test-slack", color: 'warning', message: "${env.BUILD_TAG} became unstable with change :\n" +
                "commit ${env.GIT_AUTHOR}\n" +
                "Author: ${env.CHANGE_AUTHOR_DISPLAY_NAME} <${env.CHANGE_AUTHOR_EMAIL}>\n" +
                "\t${env.GIT_COMMIT_MSG}\n" +
                "See : ${env.JOB_URL}"
            }
            failure {
                slackSend channel: "#test-slack", color: 'danger', message: "${env.BUILD_TAG} failed with change :\n" +
                "commit ${env.CHANGE_ID}\n" +
                "Author: ${env.CHANGE_AUTHOR_DISPLAY_NAME} <${env.CHANGE_AUTHOR_EMAIL}>\n" +
                "\t${env.CHANGE_TITLE}\n" +
                "See : ${env.JOB_URL}"
            }
        }
    }
}    
