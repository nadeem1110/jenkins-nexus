pipeline {
    agent any
    tools {
        maven "maven"
    }
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "10.21.34.140:8081"
        NEXUS_REPOSITORY = "java-app"
        NEXUS_CREDENTIAL_ID = "NEXUS_CRED"
    }
    stages {
        stage("Clone code from GitHub") {
            steps {
                script {
                    git branch: 'main', credentialsId: 'githubwithpassword', url: 'https://github.com/nadeem1110/jenkins-nexus';
                }
            }
        }
        
        stage("Maven Build") {
            steps {
                script {
                    sh "mvn package -DskipTests=true"
                }
            }
        }
        stage('Execute Sonarqube Report') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh "mvn sonar:sonar"
                }
            }
        }
	 stage('Building Docker Image'){
         	steps{
             		script{
              			sh "docker build . -t nadeempatel/test1:latest"
             }
         }
     }
         stage('Push Docker Image Dockerhub') {
            steps {
                script {
                 withCredentials([string(credentialsId: 'docker_id', variable: 'pass')]) {
                    sh 'docker login -u nadeempatel -p Bullet@143'
                    sh "docker push nadeempatel/test1:latest"
                 }
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
                            nexusVersion: 'nexus3',
                            protocol: 'http',
                            nexusUrl: "10.21.34.140:8081",
                            groupId: 'pom.com.mycompany.app',
                            version: 'pom.1.0-SNAPSHOT',
                            repository: 'maven-repo',
                            credentialsId: 'NEXUS_CRED',
                            artifacts: [
                                [artifactId: 'pom.my-app',
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                [artifactId: 'pom.my-app',
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
