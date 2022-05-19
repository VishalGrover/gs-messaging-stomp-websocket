pipeline{
    agent any
    parameters {
        string(name: 'NEW_VERSION', defaultValue: '')
    }
    environment {
        //Use Pipeline Utility Steps plugin to read information from pom.xml into env variables
        VERSION = readMavenPom().getVersion()
    }

    stages{
        stage('build'){
            if (${BRANCH_NAME} == 'main') {
                NEW_VERSION = ${VERSION} + "_" + ${GIT_COMMIT}
            } else {
                NEW_VERSION = ${VERSION} + "_SNAPSHOT"
            }
            agent {
                docker {
                  image 'maven:3.6.3-jdk-11-slim'
                }
            }
            steps{
                echo "Branch name is ${BRANCH_NAME}"
                echo "Version number is ${VERSION}"
                echo "${NEW_VERSION}"
                echo 'compile maven app'
                sh 'mvn compile'
            }
        }
        stage('test'){
            when {
                branch "release"
            }
            agent {
                docker {
                  image 'maven:3.6.3-jdk-11-slim'
                }
            }
            steps{
                echo 'test maven app'
                sh 'mvn clean test'
            }
        }
        stage('package'){
            when {
                branch "release"
            }
            agent {
                docker {
                  image 'maven:3.6.3-jdk-11-slim'
                }
            }
            steps{
                echo 'generating artifacts'
                sh 'mvn package -DskipTests'
                archiveArtifacts 'target/*.jar'
            }
        }
        stage('Docker Build Image and Publish') {
          when {
              branch "release"
          }
          agent any
          steps {
            script {
              docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                def dockerImage = docker.build("grovervishal591/messaging-stomp-websocket:v${env.BUILD_ID}", "./")
                dockerImage.push()
                dockerImage.push("dev")
              }
            }
          }
        }
    }
    post {
        always {
          echo 'This pipeline is completed..'
        }
    }
}