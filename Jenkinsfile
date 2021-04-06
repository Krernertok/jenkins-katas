pipeline {
  agent any
  environment {
    docker_username = 'krernertok'
  }
  stages {
    stage('Clone down') {
      agent {
        label 'swarm'
      }

      steps {
        stash(excludes: '.git', name: 'code')
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
          agent {
            docker {
              image 'gradle:jdk11'
            }
          }

          options {
            skipDefaultCheckout(true)
          }

          steps {
            unstash 'code'
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
            stash 'code'

            sh 'ls -lAh'
            deleteDir()
            sh 'ls -lAh'
          }
        }

        stage('Test app') {
          agent {
            docker {
              image 'gradle:jdk11'
            }
          }
          options {
            skipDefaultCheckout(true)
          }
          steps {
            unstash 'code'
            sh 'ci/unit-test-app.sh'
            junit 'app/build/test-results/test/TEST-*.xml'
          }
        }
      }
    }

    stage('Push Docker app') {
      when {
        beforeAgent true
        branch "master"
      }
      environment {
        DOCKERCREDS = credentials('docker_login')
      }
      options {
        skipDefaultCheckout(true)
      }
      steps {
        unstash 'code' //unstash the repository code
        sh 'ci/build-docker.sh'

        input 'Push to Docker Hub?'

        sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin' //login to docker hub with the credentials above
        sh 'ci/push-docker.sh'
      }
    }

    stage('Run component test') {
      when{
        anyOf {
          branch "master"
          changeRequest()
        }
      }
      options {
        skipDefaultCheckout()
      }
      steps {
        unstash 'code'
        sh 'ci/component-test.sh'
      }
    }
  }

  post {
    cleanup {
      deleteDir() /* clean up our workspace */
    }
  }

}