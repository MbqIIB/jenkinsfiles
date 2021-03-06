pipeline {
  agent {
    kubernetes {
      label UUID.randomUUID().toString()
      containerTemplate {
        name 'terraform'
        image 'hashicorp/terraform:latest'
        ttyEnabled true
        command 'cat'
      }
    }
  }

  options {
    buildDiscarder(logRotator(numToKeepStr: '30'))
  }

  environment {
    AWS_ACCESS_KEY_ID = credentials('aws.id')
    AWS_SECRET_ACCESS_KEY = credentials('aws.key')
  }

  stages {
    stage('Initialize') {
      steps {
        container('terraform') {
          sh 'terraform init'
        }
      }
    }
    stage('Plan') {
      steps {
        container('terraform') {
          sh 'terraform plan -out .tfplan'
        }
      }
    }
    stage('Apply') {
      when {
        expression { env.BRANCH_NAME == 'master' }
      }
      steps {
        container('terraform') {
          timeout(time: 5, unit: 'MINUTES') {
            input message: 'Continuing may change or destroy existing infrastructure, be sure to review the plan stage before continuing'
          }
          sh 'terraform apply -auto-approve .tfplan'
        }
      }
    }
  }
}
