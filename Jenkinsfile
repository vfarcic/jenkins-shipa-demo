pipeline {
  agent {
    kubernetes {
      defaultContainer "shell"
      yamlFile "jenkins-pod.yaml"
    }
  }
  environment {
    PROJECT = "jenkins-demo"
    REGISTRY_USER = "vfarcic"
  }
  stages {
    stage("Build") {
      steps {
        container("kaniko") {
          sh "/kaniko/executor --context `pwd` --destination vfarcic/jenkins-demo:latest --destination ${REGISTRY_USER}/${PROJECT}:${env.BRANCH_NAME.toLowerCase()}-${BUILD_NUMBER}"
        }
      }
    }
    stage("Test") {
      when { changeRequest target: "master" }
      steps {
        container("shipa") {
          sh "shipa app create $PROJECT-${env.BRANCH_NAME.toLowerCase()}"
          sh "shipa app deploy --app $PROJECT-${env.BRANCH_NAME.toLowerCase()} --image ${REGISTRY_USER}/${PROJECT}:${env.BRANCH_NAME.toLowerCase()}-${BUILD_NUMBER}"
        }
        container("kustomize") {
          // TODO: Uncomment
          // sh "kubectl --namespace $PROJECT-${env.BRANCH_NAME.toLowerCase()} rollout status deployment jenkins-demo"
          sh "curl http://${env.BRANCH_NAME.toLowerCase()}$PROJECT.acme.com"
        }
        container("shipa") {
          sh "shipa app remove --app $PROJECT-${env.BRANCH_NAME.toLowerCase()} --assume-yes"
        }
      }
    }
    stage("Deploy") {
      when { branch "master" }
      steps {
        container("kustomize") {
          sh "shipa app deploy --app $PROJECT --image ${REGISTRY_USER}/${PROJECT}:${env.BRANCH_NAME.toLowerCase()}-${BUILD_NUMBER}"
        }
      }
    }
  }
}