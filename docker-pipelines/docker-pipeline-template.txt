pipeline {  

    environment {  

        registry = "e.g. lylio/chatty"

        registryCredential = 'dockerhub_id'  

        dockerImage = ''  

    } 

    agent any  

    stages {  

        stage('Cloning our Git') {  

            steps {  

                git 'e.g. https://github.com/Lylio/chatty-services'

            } 

        }  

        stage('Building our image') {  

            steps {  

                script {  

                    dockerImage = docker.build registry + ":$BUILD_NUMBER"  

                } 

            }  

        } 

        stage('Deploy our image') {  

            steps {  

                script {  

                    docker.withRegistry( '', registryCredential ) {  

                        dockerImage.push()  

                    } 

                }  

            } 

        }  

        stage('Cleaning up') {  

            steps {  

                sh "docker rmi $registry:$BUILD_NUMBER"  

            } 

        }  

    } 

}
