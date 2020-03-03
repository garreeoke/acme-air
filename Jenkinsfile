pipeline {
  environment {
    registry = "garreeoke/acmenode"
    registryCredential = "dockerhub"
    label = "docker-jenkins-${UUID.randomUUID().toString()}"
    home = "/home/jenkins/agent"
    tag = "acmenode:" + "${BUILD_NUMBER}"
    repo = "garreeoke/" + "$tag"
    GIT_AUTH = credentials('garreeoke-github')
    DOCKER_AUTH = credentials('garreeoke-docker')
  }
  agent {
    kubernetes {
      yamlFile 'kubepod.yml'
    } 
  }
  stages {
    stage('Checkout Node Master') {
      steps {
        echo "Checkout Node"
        sh "mkdir ./node-master"
        dir("node-master") {
          timeout(time: 3, unit: 'MINUTES') {
            checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'garreeoke-github', url: 'https://github.com/garreeoke/node-master.git']]])
          }
        }
      }
    }
    stage ('Docker Build') {
      steps {
        dir("node-master") {
          container('docker') {
          echo "Building docker image ... $repo $DOCKER_AUTH_USR $DOCKER_AUTH_PSW"
          sh "docker build -t $repo --build-arg branch=master ."
          }
        }
      }
    }
    stage ('Docker Publish') {
      steps {
        container('docker') {
          echo "Publishing docker image"
          sh "docker login -u '$DOCKER_AUTH_USR' -p '$DOCKER_AUTH_PSW'" 
          sh "docker push $repo"
        }
      }
    }
    stage ('Update YAML') {
      steps {
        echo "Changing kubernetes yaml"
        sh "mkdir ./acme-air"
        dir("acme-air") {
          sh('''
            checkout scm
            checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'garreeoke-github', url: 'https://github.com/garreeoke/acme-air.git']]])
            sed -i -E "s/acmenode:.*/$tag/" k8s/deployment.yml
            git add deployment.yml 
            git commit -m "[Jenkins CI] updating image to acmenode:$tag"
            git push 
          ''')
        }     
      }
    }
  }
}