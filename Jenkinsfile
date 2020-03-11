pipeline {
  agent any
  options {
    buildDiscarder(logRotator(numToKeepStr: '3'))
  }
  stages {
    stage('test') {
      steps {
        slackSend(message: 'test', baseUrl: 'https://hooks.slack.com/services/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', botUser: true)
      }
    }
    stage('Sonarqube') {
        environment {
            scannerHome = tool 'sonarqube'
        }    
	steps {
             withSonarQubeEnv('sonarserver') {
                 sh "${scannerHome}/bin/sonarqube"
             }        
       }
    }	  
    stage('DockerHub') {
	agent any
        environment {
            registry = "ankit0999/docker-test"
            registryCredential = "dockerhub"
        }
       steps {
	 git 'https://github.com/Aman-chain/Hello_World_Django.git'
       }
    }	    
    stage('Build Image') {
      steps{
        script {
          dockerImage = docker.build("ankit0999/docker-test")
        }
      }
    }
    stage('Deploy Image') {
      steps{
        script {
          docker.withRegistry( '', registryCredential = "dockerhub" ) {
          sh "docker login -u ankit0999 -p admin@123"
            dockerImage.push()
	  }
	}
      }
    }
  }
}
