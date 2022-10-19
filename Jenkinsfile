pipeline {
    
    agent any
    
    environment {
        imageName = "test102"
        registryCredentials = "nexus"
        registry = "44.201.102.100:8085/"
        dockerImage = ''
    }
    
    stages {
        stage('Git Code') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/omersaif/web-app.git']]])                   }
        }
        stage('Sonarqube') {
               
             steps{
             withSonarQubeEnv(installationName: 'sonar-scanner', credentialsId: 'sonar-scanner') {
                 sh "/var/lib/jenkins/tools/sonar-scanner/bin/sonar-scanner"
                }
                
           }
        }
       stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
                
              }
            }
          }
    
    // Building Docker images
    stage('Build an image') {
      steps{
        script {
          dockerImage = docker.build imageName
        }
      }
    }

    // Uploading Docker images into Nexus Registry
    //stage('Upload To Nexus') {
     //steps{  
       //  script {
         //    docker.withRegistry( 'http://'+registry, registryCredentials ) {
           //  dockerImage.push('latest')
          //}
        //}
      //}
    //} 

    // Uploading Docker images into Nexus Registry
    stage('Image upload to Nexus') {
     steps{
         script {
             docker.withRegistry( 'http://'+registry, registryCredentials ) {
              version = VersionNumber(versionNumberString: '1.${BUILDS_ALL_TIME}')
             dockerImage.push(version)
           }
        }
      }
    }
    
    // Stopping Docker containers for cleaner Docker run
     
    stage('stop previous containers') {
         steps {
            sh 'docker ps -f name=test102 -q | xargs --no-run-if-empty docker container stop'
            sh 'docker container ls -a -fname=test102 -q | xargs -r docker container rm'
         }
       }
  
    stage('Docker Run') {
       steps{
         script {
               dockerImage.run("-p 80:80 --rm --name test102")
            }
         }
      }    
    }
}