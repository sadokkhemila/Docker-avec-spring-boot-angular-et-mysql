pipeline {
    agent any
    tools {
        maven "maven"
    }  
     environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "51.38.50.55:8081"
        NEXUS_REPOSITORY = "testproject"
        NEXUS_CREDENTIAL_ID = "nexus-cred"
        SCANNER_HOME = tool 'sonar-scanner'
    }
    
    stages {
         stage("Clone code from git") {
             steps {
                    
                    git branch:'main', url:'https://github.com/sadokkhemila/Docker-avec-spring-boot-angular-et-mysql.git',
                         credentialsId: 'githubtoken-pub'   
            }
         }
         
         stage("build and Run frontend") {
             steps {
               // Aller au répertoire du frontend
                dir('frontend') {
		script{
                    sh " docker stop front-cont || true && docker rm front-cont || true"
                    sh " docker run --name front-cont -d -p 4203:80 front-test"
                    sh "docker build -t front-test . "
		}
              }
           }
         }
        
         stage("Maven Build backend") {
             steps {
                 dir ('backend'){
			 sh "mvn package -DskipTests=true"
		 }
             }      
        }
	stage('Build Docker Image et container backend'){
	   steps {
               dir('backend'){
                script {
		            sh 'docker stop back-cont'
                            sh 'docker rm back-cont'
                            sh 'docker build -t back-test .'
		            sh 'docker run -d -p 8086:8086 --name back-cont back-test'
		       }
	       }
            }
        }
          stage('Test Qualité Sonarqube') {
               steps {           
                  withSonarQubeEnv('sonarqube-server') {
                         sh" ${SCANNER_HOME}/bin/sonar-scanner"                 
                }
            }
        }
        stage('Login and Push Images "frontend and backend" DockerHub'){
            steps {
              script {
                echo 'logging in to docker hub and pushing image..'
                withCredentials([usernamePassword(credentialsId:'dockerhub',passwordVariable:'dockerHubPassword', usernameVariable:'dockerHubUser')]) {
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                    sh "docker tag front-test:latest sadokkhemila/front-test:latest"
                    sh "docker push sadokkhemila/front-test:latest"
                    sh "docker tag back-test:latest sadokkhemila/back-test:latest"
                    sh "docker push sadokkhemila/back-test:latest"
                }
             }
            } 
        }
	stage('upload Backend to nexus'){
		steps{
		 dir('backend'){   
		  nexusArtifactUploader artifacts: [	
			         [
				      artifactId: 'backend',
				      classifier: '',
				      file: 'target/backend-0.0.1-SNAPSHOT.jar',
				      type: 'jar'	
			        ]	
		  ],
		  credentialsId: 'NEXUS_CREDENTIAL_ID,  
		  groupId: 'com.example',
		  nexusUrl: 'NEXUS_URL,
		  nexusVersion: 'NEXUS_VERSION,
		  protocol: 'NEXUS_PROTOCOL,
		  repository: 'NEXUS_REPOSITORY,
		  version: '0.0.1-SNAPSHOT'
	        }
            }
	 }
    }
}
