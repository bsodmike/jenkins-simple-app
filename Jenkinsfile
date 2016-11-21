#!groovy

properties([
  [$class: 'GithubProjectProperty', displayName: 'Simple Application', projectUrlStr: 'https://github.com/bsodmike/jenkins-simple-app/'],
  buildDiscarder(logRotator(artifactNumToKeepStr: '5', daysToKeepStr: '15'))
])


node("docker") {
  stage('Prep Build Tools img') {
    /**
     * We checkout this source directly onto the metal of the build slave, just
     * to be able to create our base image for `build-tools`.
     *
     * In the future, we can fetch this from our private registry.
     */
    checkout scm
    def tools = docker.build 'inertialbox/build-tools:snapshot'

    /**
     * From this point onwards we are operating inside the context of the
     * `build-tools` container, all nice and isolated.
     */
    tools.inside {
      stage('Checkout') {
        checkout scm
        short_commit = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        currentBuild.description = "#${short_commit}"
      }

      stage('Build') {
        sh 'mvn clean package -Dmaven.test.skip=true'
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        stash name: 'docker', includes: 'src/main/docker/Dockerfile, , target/*.jar'
      }
    }
  }

  stage('Build Docker img') {
    node("docker") {
      unstash 'docker'
      image = docker.build("inertialbox/simple-app:${short_commit}", '-f src/main/docker/Dockerfile .')
    }
  }

  stage('Validate Docker img') {
    node("docker") {
      container = image.run('-P')
      ip = container.port(8080)
    }
    try {
      input message: "Is http://${ip} ok?", ok: 'Publish'
    } finally {
      node("docker") { container.stop() }
    }
  }
}
