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
    stage('Build Preparation') {
      environment 
      {
	AWS_BIN = '/home/ec2-user/.local/bin/aws'
      }
      steps{
        script {
	  docker.withRegistry('http://930942422495.dkr.ecr.us-east-1.amazonaws.com','ecr:us-east-1:demo-ecr-credentials' )
	  {
	      sh 'docker tag aman432/docker-test:latest 930942422495.dkr.ecr.us-east-1.amazonaws.com/demo1'
              sh 'docker push 930942422495.dkr.ecr.us-east-1.amazonaws.com/demo1'
	  }
	}
      }
    }
  }
}
