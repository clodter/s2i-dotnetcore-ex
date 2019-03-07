def templateName = 'dotnet'
def project = 'dev-monolith-a314'

pipeline {
  agent any
  stages {
    stage('Build Image') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject(project) {
              openshift.selector("bc", templateName).startBuild("--wait=true")
            }
          }
        }
      }
    }
    stage('Deploy DEV') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject(project) {
              openshift.selector("dc", templateName).rollout().latest();
            }
          }
        }
      }
    }
  }
}
