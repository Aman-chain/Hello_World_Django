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
             withSonarQubeEnv('sonarqube') {
                 sh "${scannerHome}/bin/sonar-scanner"
             }        
       }
    }	  
    stage('DockerHub') {
	agent any
        environment {
            registry = "aman432/docker-test"
            registryCredential = "dockerhub"
        }
       steps {
	 git 'https://github.com/Aman-chain/Hello_World_Django.git'
       }
    }	    
    stage('Build Image') {
      steps{
        script {
          dockerImage = docker.build("aman432/docker-test")
        }
      }
    }
    stage('Deploy Image') {
      steps{
        script {
          docker.withRegistry( '', registryCredential = "dockerhub" ) {
          sh "docker login -u aman432 -p aman@1234"
            dockerImage.push()
	  }
	}
      }
    }
    stage 'Docker push'
    docker.withRegistry('https://930942422495.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:demo-ecr-credentials') {
      docker.image('demo').push('latest')
  }
  }
}


