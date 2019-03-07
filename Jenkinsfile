def templateName = 'dotnet'
def project = 'dev-monolith-a314'

pipeline {
  agent any
  stages {
    stage('Create Image Builder') {
      when {
        expression {
          openshift.withCluster() {
            openshift.withProject(project) {
              return !openshift.selector("bc", templateName).exists();
            }
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject(project) {
              openshift.newBuild("--name=${templateName}", "--image-stream=dotnet:2.1")
            }
          }
        }
      }
    }
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
    stage('Create DEV') {
      when {
        expression {
          openshift.withCluster() {
            openshift.withProject(project) {
              return !openshift.selector('dc', templateName).exists()
            }
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject(project) {
              def dc = openshift.selector("dc", templateName)
              while (dc.object().spec.replicas != dc.object().status.availableReplicas) {
                  sleep 10
              }
              openshift.set("triggers", "dc/${templateName}", "--manual")
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
