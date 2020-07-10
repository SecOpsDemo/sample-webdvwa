def SERVICE_GROUP = "sample"
def SERVICE_NAME = "webdvwa"
def IMAGE_NAME = "${SERVICE_GROUP}-${SERVICE_NAME}"
def REPOSITORY_URL = "https://github.com/SecOpsDemo/sample-webdvwa.git"
def REPOSITORY_SECRET = ""
def SLACK_TOKEN_DEV=""
def SLACK_TOKEN_DQA=""

@Library("github.com/opsnow-tools/valve-butler")
def butler = new com.opsnow.valve.v9.Butler()
def label = "worker-${UUID.randomUUID().toString()}"

properties([
  buildDiscarder(logRotator(daysToKeepStr: "60", numToKeepStr: "30"))
])
podTemplate(label: label, containers: [
  containerTemplate(name: "builder", image: "opsnowtools/valve-builder:v0.2.39", command: "cat", ttyEnabled: true, alwaysPullImage: true),
], volumes: [
  hostPathVolume(mountPath: "/var/run/docker.sock", hostPath: "/var/run/docker.sock"),
  hostPathVolume(mountPath: "/home/jenkins/.draft", hostPath: "/home/jenkins/.draft"),
  hostPathVolume(mountPath: "/home/jenkins/.helm", hostPath: "/home/jenkins/.helm")
]) {
  node(label) {
    stage("Prepare") {
      container("builder") {
        butler.prepare(IMAGE_NAME)
      }
    }
    stage("Checkout") {
      container("builder") {
        try {
          if (REPOSITORY_SECRET) {
            git(url: REPOSITORY_URL, branch: BRANCH_NAME, credentialsId: REPOSITORY_SECRET)
          } else {
            git(url: REPOSITORY_URL, branch: BRANCH_NAME)
          }
        } catch (e) {
          butler.failure(SLACK_TOKEN_DEV, "Checkout")
          throw e
        }

        butler.scan("helm")
      }
    }
    if (BRANCH_NAME == "master") {
      stage("Build Chart") {
        container("builder") {
          try {
            butler.build_chart()
          } catch (e) {
            butler.failure(SLACK_TOKEN_DEV, "Build Charts")
            throw e
          }
        }
      }
      stage("Deploy DEV") {
        container("builder") {
          try {
            butler.deploy("here", "${SERVICE_GROUP}", "${IMAGE_NAME}", "dev")
            butler.success(SLACK_TOKEN_DEV, "Deploy HERE")
          } catch (e) {
            butler.failure(SLACK_TOKEN_DEV, "Deploy HERE")
            throw e
          }
        }
      }
    }
  }
}
