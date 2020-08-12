pipeline {
  agent any
  environment {
    docker_username = 'thomasfr89'
  }
    stages {
    stage('Clone Down') {
      steps {
        stash excludes: '.git', name: 'code'
      }
    }

    stage('push docker app') {
      environment {
        DOCKERCREDS = credentials('docker_login') //use the credentials just created in this stage
      }
      steps {
        unstash 'code' //unstash the repository code
        sh 'ci/build-docker.sh'
        /* groovylint-disable-next-line LineLength */
        sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin' //login to docker hub with the credentials above
        sh 'ci/push-docker.sh'
      }
    }

    stage('Parallel execution') {
      parallel {
        stage('Say Hello') {
          steps {
            sh 'echo "hello world"'
          }
        }

        stage('Build app') {
          options {
            skipDefaultCheckout true
          }

          agent {
            docker {
              image 'gradle:jdk11'
            }
          }
          steps {
            unstash 'code'
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
              sh 'ls'
            deleteDir()
              sh 'ls'
          }
        }
      }
    }
    }
}
