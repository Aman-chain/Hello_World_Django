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
    environment 
    {
        VERSION = 'latest'
        PROJECT = 'docker-test'
        IMAGE = 'docker-test:latest'
        ECRURL = 'http://930942422495.dkr.ecr.us-east-1.amazonaws.com'
        ECRCRED = 'ecr:us-east-1:demo-ecr-credentials'
    }
  stages
  {
      stage('Build preparations')
      {
          steps
          {
              script 
              {
                  // calculate GIT lastest commit short-hash
                  gitCommitHash = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                  shortCommitHash = gitCommitHash.take(7)
                  // calculate a sample version tag
                  VERSION = shortCommitHash
                  // set the build display name
                  currentBuild.displayName = "#${BUILD_ID}-${VERSION}"
                  IMAGE = "$PROJECT:$VERSION"
              }
          }
      }
      stage('Docker build')
      {
          steps
          {
              script
              {
                  // Build the docker image using a Dockerfile
                  docker.build("$IMAGE","examples/pipelines/TAP_docker_image_build_push_ecr")
              }
          }
      }
      stage('Docker push')
      {
          steps
          {
              script
              {
                  // login to ECR - for now it seems that that the ECR Jenkins plugin is not performing the login as expected. I hope it will in the future.
                  sh("eval \$(aws ecr get-login --no-include-email | sed 's|https://||')")
                  // Push the Docker image to ECR
                  docker.withRegistry(ECRURL, ECRCRED)
                  {
                      docker.image(IMAGE).push()
                  }
              }
          }
      }
  }
    
  post
  {
      always
      {
          // make sure that the Docker image is removed
          sh "docker rmi $IMAGE | true"
      }
  }
   
  }
}


