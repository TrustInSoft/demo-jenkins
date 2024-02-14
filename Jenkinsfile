pipeline {
  agent any
  // Env variables defined in Jenkins at > Jenkins > Manage Jenkins > Configure System and Environment Variables
  // No need to define them locally in the pipeline
  // environment {
  //   TIS_VERSION = '1.46.3'
  //   TIS_PARALLEL_RUNS = '4'
  // }
  stages {
    stage('Code Checkout') {
      steps {
        checkout(
          [
            $class: 'GitSCM',
            branches: [[name: '**']],
            extensions: [[$class: 'CloneOption', noTags: false, reference: '', shallow: true]],
            userRemoteConfigs: [[url: 'https://github.com/trustinsoft/demo-jenkins']]
          ]
        )
      }
    }
    stage('Run TISA tests') {
      steps {
        script {
          echo "Run all TrustInSoft tests"
          // The number of TrustInSoft Analyzer parallel runs is controlled
          // by the Jenkins environment variable "TIS_PARALLEL_RUNS", which
          // can be set in:
          // > Jenkins > Manage Jenkins > Configure System and Environment Variables
          //
          // Note, that to run more than one test in parallel you need:
          // - the equivalent number of TrustInSoft licenses,
          // - the "parallel" tool installed on the Jenkins agent
          //   (sudo apt-get install parallel).
          sh '''#!/bin/bash
              /home/tis/${TIS_VERSION}/bin/tis-setenv.sh
              source ~/.tis.conf
              trustinsoft/run_all.sh -n ${TIS_PARALLEL_RUNS}
          '''
        }
      }
    }
    stage('Produce TISA report') {
      steps {
        script {
          echo "Produce TISA report"
          sh '''#!/bin/bash
            /home/tis/${TIS_VERSION}/bin/tis-setenv.sh
            source ~/.tis.conf
            tis-report _results
          '''
        }
      }
    }
    stage('Produce MISRA report') {
      steps {
        script {
          echo "Run tis-misra"
          // For this particular project, source files are in the "lib" directory.
          sh '''#!/bin/bash
            /home/tis/${TIS_VERSION}/bin/tis-setenv.sh
            source ~/.tis.conf
            tis-misra lib
          '''
        }
      }
    }
    stage('Publish artifacts') {
      steps {
        sh '''#!/bin/bash
            tree .
        '''
        // Artifacts of tis-report
        archiveArtifacts artifacts: 'tis_report.html', onlyIfSuccessful: false
        archiveArtifacts artifacts: '_results/**.json', onlyIfSuccessful: false
        archiveArtifacts artifacts: '_results/**.csv', onlyIfSuccessful: false
        // Artifacts of tis-misra
        archiveArtifacts artifacts: 'tis_misra_report/**', onlyIfSuccessful: false
      }
    }
    stage('Optional: Fail build if some UB were found') {
      steps {
        script {
          echo "Parse JSON report for Undefined Behaviors"
          // The below shell will return a non zero value if some UB were found
          // causing the shell to fail, and the Jenkins job to fail too (if you configure it so)
          // The below shell requires "jq" to be installed on the Jenkins agent: sudo apt install -y jq
          sh '! cat _results/*_results.json|jq "{status: .alarms.status}"|grep -H \'"status": "NOT OK"\''
        }
      }
    }
  }
}
