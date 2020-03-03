pipeline {
    environment {
        registry = "garreeoke/acmenode"
        registryCredential = "dockerhub"
        label = "docker-jenkins-${UUID.randomUUID().toString()}"
        home = "/home/jenkins/agent"
        workspace = "${home}/workspace/acmenode"
        workdir = "${workspace}/src/localhost/docker-jenkins/"
        tag = "acmenode:" + "${BUILD_NUMBER}"
        repo = "garreeoke/" + "$tag"
        GIT_AUTH = credentials('gareeoke-github')
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
            timeout(time: 3, unit: 'MINUTES') {
              checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'garreeoke-github', url: 'https://github.com/garreeoke/node-master.git']]])
            }
          }
        }
        stage ('Docker Build') {
            steps {
              container('docker') {
                echo "Building docker image ... $repo"
                sh "docker build -t $repo --build-arg branch=master ."
              }
            }
        }
        stage ('Docker Publish') {
            steps {
              container('docker') {
                sh "docker login -u $DOCKER_AUTH_USER -p $DOCKER_AUTH_PSW" 
               sh "docker push $repo"
              }
            }
        }
        stage ('Checkin') {
            steps {
                  sh('''
                     echo "USGR: $GIT_AUTH_USR"
                     git checkout --track origin/armory
                     sed -i -E "s/acmenode:.*/$tag/" deployment.yml
                     git config --global user.email "garreeoke@gmail.com"
                     git config --global user.name "garreeoke"
                     git add deployment.yml 
                     git commit -m "[Jenkins CI] Add build file"
                     git config --local credential.helper "!f() { echo username=\\$GIT_AUTH_USR; echo password=\\$GIT_AUTH_PSW; }; f"
                     git push 
                 ''')
            }
        }
      }
}