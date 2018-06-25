pipeline{

  agent any

  stages {

		stage('Checkout Submodules') {
			steps{
        echo "Checkout and init submodules"
        checkout scm: [
                $class: 'GitSCM',
                branches: scm.branches,
                doGenerateSubmoduleConfigurations: false,
                extensions: [[$class: 'SubmoduleOption',
                              disableSubmodules: false,
                              parentCredentials: false,
                              recursiveSubmodules: true,
                              reference: '',
                              trackingSubmodules: false]],
                submoduleCfg: [],
                userRemoteConfigs: scm.userRemoteConfigs
        ]
			}
		}

  }

  post{
    always {
      cleanWs()
    }
  }

}
