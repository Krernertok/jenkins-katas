pipeline {
  agent any
  environment {
    docker_username = "admin"
  }
  stages {
    stage('Clone down') {
      agent {
        label 'swarm'
      }

      steps {
        stash excludes: '.git', name: 'code'
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
            stash name: 'code'

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
      environment {
        DOCKERCREDS = credentials('docker_login')
      }

      steps {
        unstash 'code' //unstash the repository code
        sh 'ci/build-docker.sh'

        input 'Push to Docker Hub?'

        sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin' //login to docker hub with the credentials above
        sh 'ci/push-docker.sh'
      }
    }
  }

  post {
    cleanup {
      deleteDir() /* clean up our workspace */
    }
  }

}