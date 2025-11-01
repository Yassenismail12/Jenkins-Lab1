pipeline {
  agent { label 'windows' }
  options { buildDiscarder(logRotator(numToKeepStr: '3')); timestamps() }
  parameters {
    choice(name: 'ACTION', choices: ['build','test','package'], description: 'Pipeline action')
    booleanParam(name: 'VERBOSE', defaultValue: true, description: 'Extra logs')
  }
  tools { jdk 'jdk17'; maven 'maven3' }
  environment { MAVEN_OPTS = '-Dmaven.test.failure.ignore=false' }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        script { if (params.VERBOSE) echo "Branch: ${env.BRANCH_NAME ?: 'main'}" }
      }
    }
    stage('Build') {
      when { anyOf { expression { params.ACTION=='build' }; expression { params.ACTION=='package' } } }
      steps { bat 'mvn -B -V -DskipTests clean package' }
    }
    stage('Test') {
      when { anyOf { expression { params.ACTION=='test' }; expression { params.ACTION=='package' } } }
      steps {
        bat 'mvn -B test'
        junit 'target/surefire-reports/*.xml'
      }
    }
    stage('Archive') {
      when { expression { params.ACTION=='package' } }
      steps { archiveArtifacts artifacts: 'target/*.jar', fingerprint: true }
    }
  }
  post { always { cleanWs() } }
}
