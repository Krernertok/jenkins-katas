pipeline {
  agent any
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
            ci/unit-test-app.sh
            junit 'app/build/test-results/test/TEST-*.xml'
          }
        }

      }
    }

  }
}