pipeline {
  agent any
  tools {
      maven 'maven-3.9'
  }
  environment {
    DOCKER_REPO_SERVER = "891376912861.dkr.ecr.eu-central-1.amazonaws.com"
    DOCKER_REPO = "${DOCKER_REPO_SERVER}/java-maven-app"
    MANIFEST_REPO = "git@github.com:vgevorgyan009/my-task-2-infrastructure.git"
    MANIFEST_PATH = "k8s-manifests/test-deployment-1.yaml"
  }
  stages {
    stage("Read and Increment Version") {      
        steps {
            script {
                def versionFile = 'version.txt'
                def version = sh(script: "cat ${versionFile}", returnStdout: true).trim()
                def parts = version.tokenize('.')
                parts[-1] = (parts[-1].toInteger() + 1).toString()
                env.IMAGE_NAME = parts.join('.')
                sh "echo ${env.IMAGE_NAME} > ${versionFile}"
                withCredentials([usernamePassword(credentialsId: 'github-credentials-token1', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh 'git config --global user.email "jenkins@hotmail.com"'
                    sh 'git config --global user.name "Jenkins"'
                    sh "git remote set-url origin https://${USER}:${PASS}@github.com/vgevorgyan009/my-task-2-application.git"
                    sh 'git add version.txt'
                    sh 'git commit -m "Bump version to ${env.IMAGE_NAME}"'
                    sh 'git push origin main'
            }
          }
        }
    }
    stage("build app") {      
        steps {
            script {
              echo "building the application..."
              sh 'mvn clean package'
          }
      }
    }
    stage('build image and push to private repo') {
        steps {
            script {
                echo 'building the docker image...'
                withCredentials([usernamePassword(credentialsId: 'ecr-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                sh "docker build -t ${DOCKER_REPO}:${IMAGE_NAME} ."
                sh 'echo $PASS | docker login -u $USER --password-stdin ${DOCKER_REPO_SERVER}'
                sh "docker push ${DOCKER_REPO}:${IMAGE_NAME}"
    }
            }
        }
    }

   }
}
