@Library('xmos_jenkins_shared_library@v0.15.0') _

getApproval()

pipeline {
  agent {
    label 'x86_64&&brew'
  }
  environment {
    REPO = 'lib_logging'
    VIEW = getViewName(REPO)
  }
  options {
    skipDefaultCheckout()
  }
  stages {
    stage('Get view') {
      steps {
        xcorePrepareSandbox("${VIEW}", "${REPO}")
      }
    }
    stage('Library checks') {
      steps {
        xcoreLibraryChecks("${REPO}")
      }
    }
    stage('Tests') {
      steps {
        runXmostest("${REPO}", 'legacy_tests')
      }
    }
    stage('xCORE builds') {
      steps {
        dir("${REPO}") {
          xcoreAllAppsBuild('examples')
          xcoreAllAppNotesBuild('examples')
          dir("${REPO}") {
            runXdoc('doc')
          }
        }
      }
    }
  }
  post {
    success {
      updateViewfiles()
    }
    cleanup {
      xcoreCleanSandbox()
    }
  }
}

pipeline {
  agent {
    label "srv-bri-nuc2"
  }
  parameters {
    string(
      name: 'TOOLS_VERSION',
      defaultValue: '15.0.3',
      description: 'The tools version to build with (check /projects/tools/ReleasesTools/)'
    )
  }
  environment {
    // '/XMOS/tools' from get_tools.py and rest from tools installers
    TOOLS_PATH = "/XMOS/tools/${params.TOOLS_VERSION}/XMOS/xTIMEcomposer/${params.TOOLS_VERSION}"
  }
  options {
    skipDefaultCheckout()
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Install Dependencies') {
      steps {
        sh '/XMOS/get_tools.py ' + params.TOOLS_VERSION
        installDependencies()
      }
    }
    stage('Build') {
      steps {
        toolsEnv(TOOLS_PATH) {  // load xmos tools
          sh 'cd tests/debug_printf_test && ln -s ../../legacy_tests/debug_printf_test/src src && make'
        }
      }
    }
    parallel {
      stage('XS3 hardware test') {
        steps {
          dir('tests/debug_printf_test') {
            withVenv() {
              toolsEnv(TOOLS_PATH) {
                sh 'xsim test.xe &> sim_out.txt && diff sim_out.txt ../../legacy_tests/test.expect'
              }
            }
          }
        }
      }
      stage('XS3 xsim test'){
        steps {
          dir('tests/debug_printf_test') {
            withVenv() {
              toolsEnv(TOOLS_PATH) {
                sh 'xrun --io test.xe &> hw_out.txt && diff hw_out.txt ../../legacy_tests/test.expect'
              }          
            }
          }
        }
      }
    } //par
  } //stages
  post {
    cleanup {
      cleanWs()
    }
  }
}
