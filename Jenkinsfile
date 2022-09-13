pipeline {
    agent any
    
    environment{
        BUILD_USER=''
    }
    stages {
        stage('build') {
            steps {
                echo "Hello Worl!"
            }
        }
    }
    post {
        success {
            slackSend channel: "#test-slack", color: 'good', message: "${env.BUILD_TAG} became success with change :\n" +
            "commit ${env.CHANGE_ID}\n" +
            "Author: ${env.CHANGE_AUTHOR_DISPLAY_NAME} <${env.CHANGE_AUTHOR_EMAIL}>\n" +
            "\t${GIT_LAST_COMMIT_DESCRIPTION}\n" +
            "See : ${env.JOB_URL}"
        }
        unstable {
            slackSend channel: "#test-slack", color: 'warning', message: "${env.BUILD_TAG} became unstable with change :\n" +
            "commit ${env.CHANGE_ID}\n" +
            "Author: ${env.CHANGE_AUTHOR_DISPLAY_NAME} <${env.CHANGE_AUTHOR_EMAIL}>\n" +
            "\t${env.CHANGE_TITLE}\n" +
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
