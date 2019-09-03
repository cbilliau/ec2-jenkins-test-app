pipeline {
  agent {
    docker {
      image 'node:10-alpine'
      args '-p 20001-20100:3000'
    }
  }
  environment {
    CI = 'true'
    HOME = '.'
    npm_config_cache = 'npm-cache'
  }
  stages {
    stage('Install Packages') {
      steps {
        sh 'npm install'
      }
    }
    stage('Test and Build') {
      parallel {
        stage('Run Tests') {
          steps {
            sh 'npm run test'
          }
        }
        stage('Create Build Artifacts') {
          steps {
            sh 'npm run build'
          }
        }
      }
    }
    stage('Deployment') {
      parallel {
        stage('Staging') {
          when {
            branch 'staging'
          }
          steps {
            withAWS(region:'us-east-2',credentials:'7ad7e5c7-492e-446d-a66b-655fa041b2ca') {
              s3Delete(bucket: 'fas-jenkins-test-1', path:'**/*')
              s3Upload(bucket: 'fas-jenkins-test-1', workingDir:'build', includePathPattern:'**/*');
            }
            mail(subject: 'Staging Build', body: 'New Deployment to Staging', to: 'hris@freshairsensor.com')
          }
        }
        stage('Production') {
          when {
            branch 'master'
          }
          steps {
            withAWS(region:'us-east-2',credentials:'7ad7e5c7-492e-446d-a66b-655fa041b2ca') {
              s3Delete(bucket: 'fas-jenkins-test-1', path:'**/*')
              s3Upload(bucket: 'fas-jenkins-test-1', workingDir:'build', includePathPattern:'**/*');
            }
            mail(subject: 'Production Build', body: 'New Deployment to Production', to: 'chris@freshairsensor.com')
          }
        }
      }
    }
  }
}
