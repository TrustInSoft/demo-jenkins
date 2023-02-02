pipeline {
  agent any
  stages {
    stage('Code Checkout') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '**']], extensions: [[$class: 'CloneOption', noTags: false, reference: '', shallow: true]], userRemoteConfigs: [[url: 'http://github.com/trustinsoft/demo-jenkins']]])
      }
    }
    stage('Run TISA tests') {
      steps {
          // You may replace -n 1 by -n <i> if you want to run several tests in parallel 
          // To run more than 1 test in parallel you need:
          // - The equivalent number of TrustInSoft licenses
          // - To have installed the "parallel" tool on the Jenkins agent
          //   (sudo apt-get install parallel)

          // In the below cases the number of TISA parallel runs is controlled by a Jenkins
          // environment Variable (TIS_PARALLEL_RUNS) to be set in
          // Jenkins > Manage Jenkins > Configure System and Environment Variables
            sh '''#!/bin/bash
                echo "Run all TrustInSoft tests"
                /home/tis/${TIS_VERSION}/bin/tis-setenv.sh
                source ~/.tis.conf
                trustinsoft/run_all.sh -n ${TIS_PARALLEL_RUNS}
            '''
      }
    }
    stage('Produce TISA report') {
      steps {
          echo "Produce TISA report"
          sh '''#!/bin/bash
            /home/tis/${TIS_VERSION}/bin/tis-setenv.sh
            source ~/.tis.conf
            tis-report _results
          '''
      }
    }
    stage('Produce MISRA report') {
      steps {
          echo "Run tis-misra"
          // For that particular project, source files are in the "lib" directory
          sh '''#!/bin/bash
            /home/tis/${TIS_VERSION}/bin/tis-setenv.sh
            source ~/.tis.conf
            tis-misra lib
          '''
      }
    }
    stage('Publish artifacts') {
      steps {
        // Artifacts of tis-report
        archiveArtifacts artifacts: 'tis_report.html', onlyIfSuccessful: false
        archiveArtifacts artifacts: '_results/**.json', onlyIfSuccessful: false
        archiveArtifacts artifacts: '_results/**.csv', onlyIfSuccessful: false
        // Artifacts of tis-misra
        archiveArtifacts artifacts: 'tis_misra_report/**', onlyIfSuccessful: false
      }
    }
    /*
    stage('Optional: Fail build if some UB were found') {
      steps {
          echo "Parse JSON report for Undefined Behaviors"
          // The below shell will return a non zero value if some UB were found
          // causing the shell to fail, and the Jenkins job to fail too (if you configure it so)
          // The below shell requires "jq" to be installed on the Jenkins agent: sudo apt install -y jq
          sh '! cat _results/*_results.json|jq "{status: .alarms.status}"|grep -H \'"status": "NOT OK"\''
      }
    }
    */
  }
}