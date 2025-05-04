pipeline {
    agent none
    stages{

      stage('build result app'){
        agent any
        steps{
          dir('result'){
            script{
              docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
                def workerImage=docker.build("simonzn/result:v${env.BUILD_ID}", ".")
                workerImage.push()
                workerImage.push("latest")
              }
            }
          }
        }
      }

      stage('vote integration test'){
        agent any
        steps{
          dir('vote'){
            sh 'sh integration_test.sh'
          }
        }
      }

      stage('e2e test'){
        agent any
        steps{
          sh 'sh e2e.sh'
        }
      }

      stage('build vote app'){
        agent any
        steps{
          dir('vote'){
            script{
              docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
                def workerImage=docker.build("simonzn/vote:v${env.BUILD_ID}", ".")
                workerImage.push()
                workerImage.push("latest")
              }
            }
          }
        }
      }

      stage('build worker app'){
        agent any
        steps{
          dir('worker'){
            script{
              docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
                def workerImage=docker.build("simonzn/worker:v${env.BUILD_ID}", ".")
                workerImage.push()
                workerImage.push("latest")
              }
            }
          }
        }
      }

      stage('sonar analysis'){
        agent any
        when{
          branch 'master'
        }
        environment{
          sonarpath = tool 'SonarScanner'
        }
        steps{
          withSonarQubeEnv('sonar-instavote') {
            sh "${sonarpath}/bin/sonar-scanner -Dproject.settings=sonar-project.properties -Dorg.jenkinsci.plugins.durabletask.BourneShellScript.HEARTBEAT_CHECK_INTERVAL=86400"
          }
          timeout(time: 1, unit: 'HOURS') {
            // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
            // true = set pipeline to UNSTABLE, false = don't
            waitForQualityGate abortPipeline: true
          }
        }
      }

      stage('deploy to dev'){
        agent any
        when{
          branch 'master'
        }
        steps{
          sh 'docker compose up -d'
        }
      }

    }

    post{
      always{
        echo 'the job is complete'
      }
    }
}
