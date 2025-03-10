pipeline {
  agent any
  tools {
      maven 'maven-3.9'
  }
  environment {
    DOCKER_REPO_SERVER = "891376912861.dkr.ecr.eu-central-1.amazonaws.com"
    DOCKER_REPO = "${DOCKER_REPO_SERVER}/java-maven-app"
    MANIFEST_REPO = "git@github.com:vgevorgyan009/my-task-2-infrastructure.git"
    MANIFEST_PATH = "my-task-2-infrastructure/k8s-manifests/test-deployment-1.yaml"
  }
  stages {
    stage("build app") {      
        steps {
            script {
              echo "building the application..."
              sh 'mvn clean package'
          }
      }
    }
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
                    sh 'git commit -m "Bump version to new version"'
                    sh 'git push origin HEAD:main'
            }
          }
        }
    }
    stage('build image and push to private repo') {
        environment {
            AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
            AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_secret_access_key')
        }
        steps {
            script {
                echo 'building the docker image...'
                sh "aws ecr get-login-password --region eu-central-1 | docker login --username AWS --password-stdin ${DOCKER_REPO_SERVER}"
                sh "docker build -t ${DOCKER_REPO}:${IMAGE_NAME} ."
                sh "docker push ${DOCKER_REPO}:${IMAGE_NAME}"
            }
        }
    }
    stage('Update Kubernetes Manifest') {
        steps {
            script {
                echo "Updating Kubernetes manifest..."
                withCredentials([sshUserPrivateKey(credentialsId: 'k8s-manifests-repo-creds', keyFileVariable: 'SSH_KEY')]) {
                    sh """
                    rm -rf my-task-2-infrastructure
                    GIT_SSH_COMMAND="ssh -i $SSH_KEY -o StrictHostKeyChecking=no" git clone ${MANIFEST_REPO}
                    sed -i 's|image: ${DOCKER_REPO}:.*|image: ${DOCKER_REPO}:${env.IMAGE_NAME}|' ${MANIFEST_PATH}
                    cd my-task-2-infrastructure/k8s-manifests
                    git config user.email "jenkins@hotmail.com"
                    git config user.name "Jenkins"
                    git add test-deployment-1.yaml
                    git commit -m "Update image version to ${env.IMAGE_NAME}"
                    GIT_SSH_COMMAND="ssh -i $SSH_KEY -o StrictHostKeyChecking=no" git push origin main
                    """
                }
            }
        }
    }
   }
}
