pipeline {
    agent any
    tools {
        maven "maven"
    }
  
   
    
    stages {
         stage("Clone code from git") {
             steps {
                    
                    git branch:'main', url:'https://github.com/sadokkhemila/Docker-avec-spring-boot-angular-et-mysql.git',
                         credentialsId: 'githubtoken-pub'
                
            }
         }
         
         stage("build frontend") {
             steps {
               // Aller au répertoire du frontend
                dir('frontend') {
                
                    sh "docker build -t front-test . "
            
              }
           }
         }
         stage("deploy docker") {
             steps {
                script {
                    sh " docker stop front-cont || true && docker rm front-cont || true"
                    sh " docker run --name front-cont -d -p 4203:80 front-test"
                }
            }
        }
          stage('Test Qualité Sonarqube') {
              environment {
                    scannerHome = tool 'sonar-scanner';
                }
               dir('frontend') {
               steps {
                  withSonarQubeEnv('sonarqube-server') {
                         sh" ${scannerHome}/bin/sonar-scanner"
                  }
                   
                }
            }
        }
        stage('Login and Push Image DockerHub'){
            steps {
              script {
                echo 'logging in to docker hub and pushing image..'
                withCredentials([usernamePassword(credentialsId:'dockerhub',passwordVariable:'dockerHubPassword', usernameVariable:'dockerHubUser')]) {
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                    sh "docker tag front-test:latest sadokkhemila/front-test:latest"
                    sh "docker push sadokkhemila/front-test:latest"
                }
             }
            } 
        }

    }
}
