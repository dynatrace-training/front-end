pipeline {
  agent {
    label 'nodejs'
  }
  environment {
    APP_NAME = "front-end"
    ARTEFACT_ID = "sockshop/" + "${env.APP_NAME}"
    VERSION = readFile 'version'
    TAG = "10.31.240.247:5000/library/${env.ARTEFACT_ID}"
    //TAG_DEV = "${env.TAG}-${env.VERSION}-${env.BUILD_NUMBER}"
    TAG_DEV = "${env.TAG}:dev"
    TAG_STAGING = "${env.TAG}-${env.VERSION}"
  }
  stages {
    stage('Node build') {
      steps {
        checkout scm
        container('nodejs') {
          sh 'npm install'
        }
      }
    }
    stage('Docker build') {
      when {
        expression {
          return env.BRANCH_NAME ==~ 'release/.*' || env.BRANCH_NAME ==~'master'
        }
      }
      steps {
        container('docker') {
          echo "branch_name=${env.BRANCH_NAME}"

          sh "docker build -t ${env.TAG_DEV} ."
        }
      }
    }
    stage('Docker push to registry') {
      when {
        expression {
          return env.BRANCH_NAME ==~ 'release/.*' || env.BRANCH_NAME ==~'master'
        }
      }
      steps {
        container('docker') {
          sh "docker push ${env.TAG_DEV}"
        }
      }
    }
    stage('Deploy to dev namespace') {
      when {
        expression {
          return env.BRANCH_NAME ==~ 'release/.*' || env.BRANCH_NAME ==~'master'
        }
      }
      steps {
        container('kubectl') {
          sh "kubectl -n dev apply -f manifest/front-end.yml"
        }
      }
    }
    stage('Mark artifact for staging namespace') {
      when {
        expression {
          return env.BRANCH_NAME ==~ 'release/.*'
        }
      }
      steps {
        container('docker'){
          sh "docker tag ${env.TAG_DEV} ${env.TAG_STAGING}"
          sh "docker push ${env.TAG_STAGING}"
        }
      }
    }
    
    stage('Deploy to staging') {
      when {
        expression {
          return env.BRANCH_NAME ==~ 'release/.*'
        }
      }
      agent {
        label 'git'
      }
      steps {
        echo "update sockshop deployment yaml for staging -> github webhook triggers deployment to staging"
        echo "apply sockshop deployment yaml to staging environment"
      }
    }
    
  }
}