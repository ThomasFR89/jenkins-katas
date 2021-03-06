pipeline {
  agent any
  environment {
    docker_username = 'thomasfr89'
  }
  stages {
    stage('Clone Down') {
      steps {
        stash excludes: '.git', name: 'code'
        deleteDir()
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
            stash excludes: '.git', name: 'code'
              sh 'ls'
            deleteDir()
              sh 'ls'
          }
        }
        stage('Test') {
          when { }
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
            sh 'ci/unit-test-app.sh'
            junit 'app/build/test-results/test/TEST-*.xml'
          }
        }
      }
    }

    stage('push docker app') {
      when {
        beforeAgent true
        branch 'master'
      }
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

    stage('component test') {
      when {
        not {
          branch 'development'
        }
      }
      options {
        skipDefaultCheckout(true)
      }
      steps {
        unstash 'code'
        sh 'ci/component-test.sh'
      }
    }
  }
}
