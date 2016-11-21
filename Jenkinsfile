#!groovy

properties([
  [$class: 'GithubProjectProperty', displayName: 'Simple Application', projectUrlStr: 'https://github.com/bsodmike/jenkins-simple-app/'],
  buildDiscarder(logRotator(artifactNumToKeepStr: '5', daysToKeepStr: '15'))
])

node("docker") {
  docker.image('ubuntu:latest').inside {
    stage('Checkout') {
      checkout scm
      short_commit = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
      currentBuild.description = "#${short_commit}"
    }
  }
}
