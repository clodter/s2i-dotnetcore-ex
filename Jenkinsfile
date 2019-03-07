def templateName = 'dotnet' 

pipeline {
  agent {
    node {
      label 'dotnet' 
    }
  }
  options {
    timeout(time: 20, unit: 'MINUTES') 
  }
  stages {
    stage('preamble') {
        steps {
            script {
                openshift.verbose()
                openshift.withCluster() {
                    openshift.withProject('dev-monolith-a314') {
                        echo "Using project: ${openshift.project('dev-monolith-a314')}"
                    }
                }
            }
        }
    }

    stage('build') {
      steps {
        script {
            openshift.verbose()
            openshift.withCluster() {
                openshift.withProject('dev-monolith-a314') {
                  def builds = openshift.selector("bc", templateName).related('builds')
                  timeout(5) { 
                    builds.untilEach(1) {
                      return (it.object().status.phase == "Complete")
                    }
                  }
                }
            }
        }
      }
    }

    stage('deploy') {
      steps {
        script {
            openshift.verbose()
            openshift.withCluster('dev-monolith-a314') {
                openshift.withProject() {
                  def rm = openshift.selector("dc", templateName).rollout()
                  timeout(5) { 
                    openshift.selector("dc", templateName).related('pods').untilEach(1) {
                      return (it.object().status.phase == "Running")
                    }
                  }
                }
            }
        }
      }
    }

    stage('tag') {
      steps {
        script {
            openshift.verbose()
            openshift.withCluster('dev-monolith-a314') {
                openshift.withProject() {
                  openshift.tag("${templateName}:latest", "${templateName}-staging:latest") 
                }
            }
        }
      }
    }
  }
}
