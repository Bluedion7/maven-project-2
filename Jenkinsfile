pipeline {
  agent any
  environment {
    WORKSPACE = "${env.WORKSPACE}"
  }
  tools {
    maven 'localMaven'
    jdk 'localJdk'
  }
  stages {
    stage('Build') {
      steps {
        sh 'mvn clean package'
      }
      post {
        success {
          echo ' now Archiving '
          archiveArtifacts artifacts: '**/*.war'
        }
      }
    }
    stage('SonarQube Scan') {
      steps {
        sh """mvn sonar:sonar \
  -Dsonar.host.url=http://44.201.154.104:9000 \
  -Dsonar.login=5e64af7e4c29d5e004e51a9ea51a81e62891062a"""
      }
    }
    stage('Upload to Artifactory') {
      steps {
        sh "mvn clean deploy -DskipTests"
      }

    }
    stage('Deploy to DEV') {
      environment {
        HOSTS = "dev"
      }
      steps {
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }

    }
    stage('Approval for stage') {
      steps {
        input('Do you want to proceed?')
      }
    }
    stage('Deploy to Stage') {
      environment {
        HOSTS = "dev"
      }
      steps {
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }

    }
    stage('Approval') {
      steps {
        input('Do you want to proceed?')
      }
    }
    stage('Deploy to PROD') {
      environment {
        HOSTS = "prod"
      }
      steps {
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
    }
  }
}

