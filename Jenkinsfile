pipeline{

  agent any

  stages {

		stage('Checkout Submodules') {
			steps{
        sh 'git submodule update'
        //checkout scm: [
        //        $class: 'GitSCM',
        //        branches: scm.branches,
        //        doGenerateSubmoduleConfigurations: false,
        //        extensions: [[$class: 'SubmoduleOption',
        //                      disableSubmodules: false,
        //                      parentCredentials: false,
        //                      recursiveSubmodules: true,
        //                      reference: '',
        //                      trackingSubmodules: false]],
        //        submoduleCfg: [],
        //        userRemoteConfigs: scm.userRemoteConfigs
        //]
			}
		}

    stage('Updating submodules'){
      steps{
        sh 'git submodule update'
      }
    }

  }

  post{
    always {
      cleanWs()
    }
  }

}
