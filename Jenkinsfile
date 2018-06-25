pipeline{

  agent any

  stages {

		stage('Checkout Submodules') {
			steps{
        //sh 'git submodule update'
        sh 'pwd'
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
        sh 'git submodule update --init --recursive'
      }
    }

    stage('Making ls'){
      steps{
        sh 'ls -la'
        sh 'ls -la themes/themes'
        sh 'ls public'
      }
    }

  }

  post{
    always {
      cleanWs()
    }
  }

}
