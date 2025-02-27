pipeline {
  agent none

  tools{
    maven 'maven 3.6.1'
  }
  
  
  stages{

    stage('worker build'){
      when{
        changeset "**/worker/**"
      }
      agent{
        docker{
          image 'maven:3.6.1-jdk-8-slim'
          args '-v $HOME/.m2:/root/.m2'
        }
      }
      steps{
        echo 'Compiling worker app'
        dir('worker'){
          sh 'mvn compile'
        }
      }
    }
    
    stage('worker-test'){
      when{
          changeset "**/worker/**"
      }
      agent{
        docker{
          image 'maven:3.6.1-jdk-8-slim'
          args '-v $HOME/.m2:/root/.m2'
        }
      }
      steps{
        echo 'Running Unit Tets on worker app'
        dir('worker'){
          sh 'mvn clean test'
        }
      }
    }
    
    stage('worker-package'){
      when{
          changeset "**/worker/**"
      }
      
      agent{
        docker{
          image 'maven:3.6.1-jdk-8-slim'
          args '-v $HOME/.m2:/root/.m2'
        }
      }
      
      steps{
        echo 'Packaging worker app'
        dir('worker'){
          sh 'mvn package -DskipTests'
          archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
        }
      }
    }
    
    stage('worker-docker-package'){
        agent any
        when{
          changeset "**/worker/**"
          branch 'master'
        }
        steps{
          echo 'Packaging worker app with docker'
          script{
            docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                def workerImage = docker.build("khaerulanam/worker:v${env.BUILD_ID}", "./worker")
                workerImage.push()
                workerImage.push("${env.BRANCH_NAME}")
                workerImage.push("latest")
            }
          }
        }
    }

    stage('result-build'){
      agent {
        docker{
          image 'node:8.16.0-alpine'
        }
      }
      when{
        changeset "**/result/**"
      }
      steps{
        echo 'Compiling result app'
        dir('result'){
          sh 'npm install'
        }
      }
    }
    
    stage('result-test'){
      agent {
        docker{
          image 'node:8.16.0-alpine'
        }
      }
      when{
          changeset "**/result/**"
      }
      steps{
        echo 'Running Unit Tets on result app'
        dir('result'){
          sh 'npm install'
          sh 'npm test'
        }
      }
    }
    
    stage('result-docker-package'){
        agent any
        when{
          changeset "**/result/**"
          branch 'master'
        }
        steps{
          echo 'Packaging result app with docker'
          script{
            docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                def workerImage = docker.build("khaerulanam/result:v${env.BUILD_ID}", "./result")
                workerImage.push()
                workerImage.push("${env.BRANCH_NAME}")
                workerImage.push("latest")
            }
          }
        }
    }

    stage('vote-build'){
      agent {
        docker{
          image 'python:2.7.16-slim'
          args '--user root'
        }
      }
      when{
        changeset "**/vote/**"
      }
      steps{
        echo 'Compiling vote app'
        dir('vote'){
          sh 'pip install --upgrade pip'
          sh 'pip install -r requirements.txt'
        }
      }
    }
    
    stage('vote-test'){
      agent {
        docker{
          image 'python:2.7.16-slim'
          args '--user root'
        }
      }
      when{
          changeset "**/vote/**"
      }
      steps{
        echo 'Running Unit Tets on vote app'
        dir('vote'){
          sh 'pip install --upgrade pip'
          sh 'pip install -r requirements.txt'
          // sh 'which nosetest'
          sh 'nosetests -v'
        }
      }
    }
    
    stage('vote-docker-package'){
        agent any
        when{
          changeset "**/vote/**"
          branch 'master'
        }
        steps{
          echo 'Packaging vote app with docker'
          script{
            docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                def workerImage = docker.build("khaerulanam/vote:v${env.BUILD_ID}", "./vote")
                workerImage.push()
                workerImage.push("${env.BRANCH_NAME}")
                workerImage.push("latest")
            }
          }
        }
    }

  }
  post{
    always{
      echo 'Building multibranch pipeline for worker is completed..'
    }
    failure {
      slackSend (channel: "instavoteg2", message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
    }
    success {
      slackSend (channel: "instavoteg2", message: "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
    }
  }
}
