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
            checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'garreeoke-github-token', url: 'https://github.com/garreeoke/node-master.git']]])
          }
        }
      }
    }
    stage ('Docker Build') {
      steps {
        dir("node-master") {
          container('docker') {
          echo "Building docker image ... $repo $DOCKER_AUTH_USR $DOCKER_AUTH_PSW"
          sh "docker build --no-cache -t $repo --build-arg branch=master ."
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
        echo "Checkout Spin-apps ..."
        sh "mkdir ./spin-apps"
        dir("${env.WORKSPACE}/spin-apps") {
          timeout(time: 3, unit: 'MINUTES') {
            checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'garreeoke-github-token', url: 'https://github.com/garreeoke/spin-apps.git']]])
          }
          sh('''
            git checkout --track origin/master
            sed -i -E "s/acmenode:.*/$tag/" acmeair/acme-air-dep.yml
            sed -i -E "s/jenkinsbuild: .*/jenkinsbuild: \'${BUILD_NUMBER}\'/" acmeair/acme-air-dep.yml
            git config --global user.email "garreeoke@gmail.com"
            git config --global user.name "$GIT_AUTH_USR"
            git add acmeair/acme-air-dep.yml
            git commit -m "[Jenkins CI] updating image to acmenode:$tag"
            git config --local credential.helper "!f() { echo username=\\$GIT_AUTH_USR; echo password=\\$GIT_AUTH_PSW; }; f"
            git push 
          ''') 
        }    
      }
    }
  }
}
