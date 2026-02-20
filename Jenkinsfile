pipeline {
   agent any
   options { skipDefaultCheckout(true) }

   stages {
      stage('Get Code') {
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

      stage('Static') {
         steps {
            sh 'whoami'
            echo WORKSPACE
            sh 'hostname'
            sh 'flake8 --exit-zero --format=pylint src > flake8.out'
            recordIssues(
               tools: [flake8(name: 'Flake8.out', pattern: 'flake8.out')])
         }
      }

      stage('Security') {
         steps {
            sh 'whoami'
            echo WORKSPACE
            sh "pwd && ls -lah"
            sh 'hostname'
            sh 'bandit --recursive src/ --format custom --output bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"'
            recordIssues(
               sourceCodeRetention: 'LAST_BUILD',
               tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')])
         }
      }

      stage('Deploy'){
         steps {
            script {
               env.BUCKET_NAME = sh(
                  script: 'echo jenkins-sam-bucket-$(date +%s)',
                  returnStdout: true
               ).trim()
            }
            echo "$BUCKET_NAME"
            sh '''
               pwd
               sam build
               sam validate --region us-east-1

               aws s3 mb s3://$BUCKET_NAME --region us-east-1
            
               sam deploy \
                  --config-env staging \
                  --s3-bucket $BUCKET_NAME
            '''
         }
      }

      stage('Rest Test'){
         steps {
            script{
               env.BASE_URL = sh(
                  script: "aws cloudformation describe-stacks --stack-name todo-list-aws-staging --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' --region us-east-1 --output text",
                  returnStdout: true).trim()
            }

            echo "$BASE_URL"
            sh 'printenv'
            sh '''
               export BASE_URL=$BASE_URL
               export PYTHONPATH=$PYTHONPATH:$WORKSPACE
               pytest --junitxml=result-integration.xml ./test/integration/todoApiTest.py
            '''
            junit 'result-integration.xml'

            sh '''
               aws s3 rb s3://$BUCKET_NAME --force
            '''
         }
      }

      stage('Promote'){
         steps{
            sh '''
               git checkout master
               git config merge.ours.driver true
               git merge origin/develop
               git push origin master
            '''
         }
      }
   }
}
