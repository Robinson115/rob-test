def getBuildUser(){
    return currentBuild.rawBuild.getCause(Cause.UserIdCause).getUserId()
}
pipeline {
    "agent any"{
        kubernetes {
        	yaml '''
apiVersion: v1
kind: Pod
spec:
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
  - name: maven
    image: maven:3.6.3-jdk-11-slim
    command:
    - cat
    tty: true
    volumeMounts:
      - mountPath: "/root/.m2"
        name: m2
  - name: aws
    image: mesosphere/aws-cli
    command:
    - cat
    tty: true
  volumes:
  - name: dockersock
    hostPath:
      path: /var/run/docker.sock
  - name: m2
    persistentVolumeClaim:
      claimName: m2
  - name: docker-config
    configMap:
      name: docker-config
'''
            defaultContainer 'docker'
        }
    }
    environment { 
        ECR_URL = 'https://310584731584.dkr.ecr.ap-south-1.amazonaws.com'
        dockerImage = ''
        BUILD_USER=''

    }
    stages {
        stage('env') {
            steps {
                echo "Generating Variables"
                script{
                    String[] str = env.BRANCH_NAME.split("/");
                    env.ENV_NAME = str.length>1?str[1]:str[0];
                    env.TAG_NAME = env.ENV_NAME=="master"?'latest':env.ENV_NAME;
                    env.REPO_NAME = env.GIT_URL.substring(env.GIT_URL.lastIndexOf('/') + 1, env.GIT_URL.lastIndexOf('.'))
                }
                echo "Printing Variables"
                echo "ENV_NAME 	: $ENV_NAME"
                echo "TAG_NAME 	: $TAG_NAME"
                echo "REPO_NAME : $REPO_NAME"
            }
        }
        stage ('parallel Stage') {
            parallel {
                stage ('infra') {
                    steps {
                    	echo "Triggering Infra Branch Build"
                        build job: 'check_and_setup_lower_env_infra', parameters: [ string(name: 'ENV_NAME', value: "$ENV_NAME") ]
                        echo "Infra Branch Build Completed"
                    }
                }
                
                stage ('Application') {
                    stages {
                        stage('build') {
                        	steps {
	                        	container('maven') {                        		
	                            	echo 'Build maven'
	                            	sh script: '''
	                            	chmod +x build.sh
	                            	./build.sh
	                            	''' 
	                        	}
	                        }
                        }
            
                        stage('containerize') {
                            steps {
                            echo 'Make docker image'
                                script{
                                    dockerImage = docker.build("$REPO_NAME")
                                }
                            }
                        }
            
                        stage('publish') {
                            steps {
                                script{
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
			                ./kubectl rollout restart deployment hotels
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
    post {
        always{
            script {
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
