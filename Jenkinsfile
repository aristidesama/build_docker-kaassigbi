pipeline{

    agent any

    environment {
        BRANCH_NAME = "${GIT_BRANCH.split("/")[1]}"
    }

    stages{
        stage('Echo') {
            steps{
                sh "echo Je suis ici !"
                }
            }
        stage('Show Release') {
            steps {
                sh "cat /etc/os-release"
            }
        }
        stage('Initialisation') {
            steps {
                sh "make clean"
                sh "make venv" 
                sh "make install"
            }
        }

        stage('Unit Test') {
            steps {
                sh "make test"
            }
        }

        stage('Build docker image') {
            steps {
                sh "docker ps -a" 
                sh "echo $BRANCH_NAME"
                sh "echo $env.GIT_BRANCH"
                sh "printenv"
                sh "docker build -t aristidesama2/${BRANCH_NAME}_image ."
            }
        }


        stage('Test docker image') { 
            steps {
                sh "docker run -d -p 5000:8000 --name default_container_$env.BRANCH_NAME aristidesama2/${BRANCH_NAME}_image"
            }
        }

                
        stage('Cleanup ') {
            steps {
                sh "docker container stop default_container_$env.BRANCH_NAME"
                sh "docker container rm default_container_$env.BRANCH_NAME" 
            }
        }

        stage("Push image to Docker Hub"){
            steps{
                withCredentials([usernamePassword(credentialsId: 'DOCKER_CREDS_KAASSIGBI', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]){
                    sh """
                    docker logout  && \
                    docker login  --username $USERNAME --password $PASSWORD && docker push aristidesama2/${BRANCH_NAME}_image:latest
                    """
                }
            }
        }
    } 

    post{
        always {
            
            sh (returnStdout: true, script: '''#!/bin/bash 
            if [ "$(docker ps -a -q -f name=default_container_$env.BRANCH_NAME)" ]; then 
            docker container stop default_container_$env.BRANCH_NAME && \
            docker container rm default_container_$env.BRANCH_NAME
            echo "EXIST and DELETE";
            fi
            '''.stripIndent())
            
            }
    }    
}
