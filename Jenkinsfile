pipeline {
   agent any
   options { skipDefaultCheckout(true) }
   environment {
      GIT_TOKEN = credentials('GIT_HUB_TOKEN')
   }

   stages {
      stage('Get Code') {
        steps {
            checkout scmGit(
               branches: [[ name: "master" ]],
               userRemoteConfigs: [[ url: 'https://github.com/Gaizka-Dev/todo-list-aws' ]],
               credentialsID: 'GIT_TOKEN'
            )
            sh 'whoami'
            echo WORKSPACE
            sh 'hostname'
        }
      }
   }

   post {
      cleanup {
         deleteDir()
      }
   }
}
