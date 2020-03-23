#!/usr/bin/env groovy
pipeline {
  agent any
  tools {nodejs "PipelineNode"}
  stages {
    stage('Install VBT, SFDX-CLI and vlocityestools ') {
      steps {
      	// Step to Install and Setup VBT and SFDX-CLI
		    sh 'node -v'
		    sh 'npm install -g vlocity sfdx-cli'
		    sh 'npm install -g sfdx-cli'
        sh 'sfdx plugins:install vlocityestools'
      }
    }
    stage('SFDX-Auth') {
      steps {
      	// creating SFDX Alias for auth
		    sh 'echo ${SFDX_URL}'
		    sh 'echo ${SFDX_URL} > env.sfdx'
		    sh 'sfdx force:auth:sfdxurl:store -d -a ${SFDX_URL} -f env.sfdx'
		    sh 'rm -rf env.sfdx'
		    sh 'sfdx force:org:display -u ${SFDX_URL}'
      }
    }
    stage('SF Deploy') {
      steps {
        script {
          def folder = new File( './salesforce_sfdx_delta' )
          // Remove ols SF Delta Folder
          if (folder.exists()) {
            sh 'rm -rf salesforce_sfdx_delta'
          } 
      	  //Create delta SF Folder
          sh 'sfdx vlocityestools:sfsource:createdeltapackage -u ${SFDX_URL} -p cmt -d salesforce_sfdx'
          sh 'ls -la'
          // SF Metadata Deploy - Only if delta Package Exits
          if (folder.exists()) {
            sh 'echo "### SF DELTA-FOLDER FOUND - Deploying now..."'
            sh 'sfdx force:source:deploy --sourcepath salesforce_sfdx_delta --targetusername ${SFDX_URL} --verbose'
          } else {
            sh 'echo "### NO SF DELTA-FOLDER FOUND"'
          }
        }
      }
    }
    stage('Vlocity Deploy') {
      steps {
      	// VBT Deploy
		    sh 'vlocity -sfdx.username ${SFDX_URL} -job Deploy_Delta.yaml packDeploy --verbose true --simpleLogging true'
        // Apex Post Deplyment Jobs (Optional)
		    sh 'vlocity -sfdx.username ${SFDX_URL} --nojob runApex -apex apex/RunProductBatchJobs.cls --verbose true --simpleLogging true'
      }
    }
  }  
}