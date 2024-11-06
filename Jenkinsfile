pipeline {
  agent {
    docker {
      image 'maven:3.8-openjdk-17-slim'
    }

  }
  stages {
    stage('build') {
      steps {
        echo 'compile maven app'
        sh 'mvn compile'
      }
    }

    stage('test') {
      steps {
        echo 'test maven app'
        sh 'mvn clean test'
      }
    }

    stage('package') {
      parallel {
        stage('package') {
          steps {
            echo 'package maven app'
            sh '''# Truncate the GIT_COMMIT to the first 7 characters
GIT_SHORT_COMMIT=$(echo $GIT_COMMIT | cut -c 1-7)
# Set the version using Maven
mvn versions:set -DnewVersion="$GIT_SHORT_COMMIT"
mvn versions:commit'''
            sh 'mvn package -DeskipTests'
            archiveArtifacts 'target/*.jar'
          }
        }

        stage('Docker B&P**.') {
          agent any
          steps {
            script {
              docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                def commitHash = env.GIT_COMMIT.take(7)
                def dockerImage = docker.build("murtajakadiyani/sysfoo:${commitHash}", "./")
                dockerImage.push()
                dockerImage.push("latest")
                dockerImage.push("dev")
              }
            }

          }
        }

      }
    }

  }
  tools {
    maven 'Maven-3.9.9'
  }
}