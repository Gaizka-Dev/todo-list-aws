pipeline {
   agent any
   options { skipDefaultCheckout(true) }

   stages {
      stage('Get Code') {
        agent {label 'agente1'}
        steps {
            checkout scmGit(
               branches: [[ name: "develop" ]],
               extensions: [ cloneOption( shallow: true, depth: 1 )],
               userRemoteConfigs: [[ url: 'https://github.com/Gaizka-Dev/todo-list-aws' ]]
            )
            sh 'whoami'
            echo WORKSPACE
            sh 'hostname'
            stash name:'code', includes:'**'
        }
      }
   }
}
