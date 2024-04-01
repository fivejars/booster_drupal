import groovy.json.*

pipeline {
  agent {
    node {
      label 'slave'
      customWorkspace 'builds/dskit/dskit-' + env.BRANCH_NAME
    }
  }
  environment {
    VIRTUAL_HOST = "${BRANCH_NAME.toLowerCase()}.dskit.${VIRTUAL_HOST_BASE}"
    BUILD_URL = "http://${VIRTUAL_HOST}/"
    COMMENT_URL = "https://api.github.com/repos/fivejars/drupal_starterkit/issues/${CHANGE_ID}/comments"
  }
  stages {
      stage('Checkout') {
          steps {
              sh 'chmod -R +w ./* && rm -rf ./* || true'
              checkout poll: false,
                  scm: [
                      $class: 'GitSCM',
                      branches: [[name: '*/master']],
                      extensions: [
                          [$class: 'CheckoutOption']
                      ],
                      userRemoteConfigs: [[
                          url: 'https://github.com/fivejars/drupal_starterkit_docksal.git'
                      ]]
                  ]
              sh 'echo "VIRTUAL_HOST=\"$VIRTUAL_HOST\"" > .docksal/docksal-local.env'
              sh 'echo "DISTRIBUTION_VERSION=\"$CHANGE_BRANCH\"" >> .docksal/docksal-local.env'
          }
      }
      stage('build') {
          steps {
              sh 'fin build_project'
              sh 'fin init'
          }
      }
  }
  post {
    success {
      script {
        def body = """\
Successfully built!
--------------

| Environment   | Link          |
| ------------- | ------------- |
| Build         | ${BUILD_URL}  |

--------------
Refer to this link for build results (access rights to CI server needed):
${RUN_DISPLAY_URL}"""

        def jsonbody = JsonOutput.toJson([body: body])
        def response = httpRequest authentication: 'githubfjbot', httpMode: 'POST', requestBody: jsonbody, responseHandle: 'STRING', url: COMMENT_URL
        if (response.status != 201) {
          error("Failed creating comment: " + response.status)
        }
      }
    }
    failure {
      script {
        def body = """\
Failure!
--------------

Refer to this link for build results (access rights to CI server needed):
${RUN_DISPLAY_URL}"""

        def jsonbody = JsonOutput.toJson([body: body])
        httpRequest authentication: 'githubfjbot', httpMode: 'POST', requestBody: jsonbody, responseHandle: 'STRING', url: COMMENT_URL
      }
    }
  }
}
