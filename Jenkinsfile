
def COLOR_MAP = [
    'STARTED': 'warning'
    'SUCCESS': 'good',
    'FAILURE': 'danger'
]

def getBuildUser(){
    return currentBuild.rawBuild.getCause(Cause.UserIdCause).getUserId()
}

pipeline {
    agent {

    environment{
        BUILD_USER=''
    }
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            spec:
              nodeSelector:
                node-class: jenkins
              containers:
              - name: docker
                image: docker
                command:
                - cat
                tty: true
                volumeMounts:
                - name: dockersock
                  mountPath: /var/run/docker.sock
                - name: docker-config
                  mountPath: /root/.docker
              - name: node
                image: node:14.10-alpine
                command:
                - cat
                tty: true
              - name: aws
                image: mesosphere/aws-cli
                command:
                - cat
                tty: true
              volumes:
              - name: dockersock
                hostPath:
                  path: /var/run/docker.sock
              - name: docker-config
                configMap:
                  name: docker-config
              tolerations:
              - key: "type"
                operator: "Equal"
                value: "jenkins"
                effect: "NoSchedule"     
            '''
            defaultContainer 'docker'
        }
    }
    environment {
        ECR_URL = 'https://310584731584.dkr.ecr.ap-south-1.amazonaws.com'
        dockerImage = ''
    }
    stages {
        stage('Environment Variables') {
            steps {
                echo 'Generating Variables'
                script {
                    String[] str = env.BRANCH_NAME.split('/')
                    env.ENV_NAME = str.length > 1 ? str[1] : str[0]
                    env.TAG_NAME = env.ENV_NAME == 'master' ? 'latest' : env.ENV_NAME
                    env.REPO_NAME = env.GIT_URL.substring(env.GIT_URL.lastIndexOf('/') + 1, env.GIT_URL.lastIndexOf('.')).toLowerCase()
                }
                echo 'Printing Variables'
                echo "ENV_NAME : $ENV_NAME"
                echo "TAG_NAME : $TAG_NAME"
                echo "REPO_NAME : $REPO_NAME"
            }
        }
        stage('Setup Infra / Build and Containerize and Publish') {
            parallel {
                stage ('Infra') {
                    steps {
                        echo 'Triggering Infra Branch Build'
                        build job: 'check_and_setup_lower_env_infra', parameters: [ string(name: 'ENV_NAME', value: "$ENV_NAME") ]
                        echo 'Infra Branch Build Completed'
                    }
                }

                stage ('Build and Containerize and Publish') {
                    stages {
                        stage('Build') {
                            steps {
                                container('node') {
                                    echo 'Building FE'
                                    sh script: '''
                                        chmod +x build.sh
                                        ./build.sh
                                    '''
                                }
                            }
                        }

                        stage('Containerize') {
                            steps {
                                echo 'Building docker image'
                                script {
                                    dockerImage = docker.build("$REPO_NAME")
                                }
                            }
                        }

                        stage('Publish') {
                            steps {
                                script {
                                    sh 'wget https://amazon-ecr-credential-helper-releases.s3.us-east-2.amazonaws.com/0.3.1/linux-amd64/docker-credential-ecr-login'
                                    sh 'chmod +x docker-credential-ecr-login'
                                    sh 'mv docker-credential-ecr-login /bin/docker-credential-ecr-login'
                                    docker.withRegistry("${env.ECR_URL}/${env.REPO_NAME}") {
                                        dockerImage.push("$TAG_NAME")
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }

        stage('deploy') {
            steps {
                container('aws') { 
                    script {
                        if (env.BRANCH_NAME == 'master') {
                            echo 'Deploying to master'
                            sh script: '''
                            apk add --update curl
                            curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
                            chmod +x ./kubectl
                            ./kubectl apply -f ./k8s.yaml
                            ./kubectl rollout restart deployment plato
                            '''
                        } else {
                            withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'wwmib', keyFileVariable: 'pemFile')]) {
                                echo "Deploying to master ${env.ENV_NAME}"
                                sh script: '''
                                #!/bin/sh
                                IP=$(aws ec2 describe-instances \
                                    --filters "Name=tag:Name,Values=$ENV_NAME" --region 'ap-south-1' \
                                    --query 'Reservations[].Instances[].PrivateIpAddress' --output text)
                                DOCKER_TAG_NAME=$(echo ${REPO_NAME}_TAG | tr [a-z] [A-Z]| tr - _)
                                echo "DOCKER_TAG_NAME = ${DOCKER_TAG_NAME}"
                                apk add --update openssh
                                ssh -o StrictHostKeyChecking=no -i ${pemFile} ubuntu@${IP} <<EOF
                                    cd /home/ubuntu/docker
                                    sed -i 's/ENV_NAME=.*/ENV_NAME='"${ENV_NAME}"'/' .env
                                    sed -i 's/'"${DOCKER_TAG_NAME}"'=.*/'"${DOCKER_TAG_NAME}"'='"${ENV_NAME}"'/' .env
                                    docker-compose stop ${REPO_NAME}
                                    docker-compose pull
                                    docker-compose up -d
                                '''
                            }
                        }
                    }
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
                      color: COLOR_MAP[currentBuild.currenResult],
                      message: "*${currentBuild.currentResult}:* ${env.JOB_NAME} build ${env.BUILD_NUMBER} by ${BUILD_USER} \n More information at: $(env.BUILD_URL}"
        }
    }
}
