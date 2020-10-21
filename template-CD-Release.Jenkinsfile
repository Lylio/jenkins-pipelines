pipeline {
    agent {
        label "master"
    }
    tools {
        maven "Maven"
    }
    environment {
        //NEXUS
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "http://http://54.154.221.150:8081/"
        NEXUS_REPOSITORY = "maven-nexus-repo"
        NEXUS_CREDENTIAL_ID = "nexus-user-credentials"
        //DOCKER
        registry = "lylio/chatty"
        registryCredential = 'dockerhub_id'
        dockerImage = ''
    }
    stages {
        stage("Clone code from GitHub") {
            steps {
                script {
                    git *** 'URL INCLUDING .git SUFFIX' ***;
                }
            }
        }
        stage('Building Docker image') {
            steps {
                script {
                     dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
       stage('Deploy Docker image') {
            steps {
                script {
                     docker.withRegistry( '', registryCredential ) {
                        dockerImage.push()
                     }
                }
            }
       }
       stage('Cleaning Docker up') {
            steps {
                sh "docker rmi $registry:$BUILD_NUMBER"
            }
       }
       stage("Maven Build") {
            steps {
                script {
                    sh "mvn package -DskipTests=true"
                }
            }
        }
        stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
    }
}
