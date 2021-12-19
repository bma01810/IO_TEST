pipeline {
  agent any

  environment {
    IO_POC_PROJECT_NAME = 'bma_IO-POC-insecure-bank'
    IO_POC_PROJECT_VERSION = "1.0"
    POLARIS_ACCESS_TOKEN = credentials('polaris-token')
    BLACKDUCK_ACCESS_TOKEN = credentials('BlackDuck-AuthToken')
    IS_SAST_ENABLED= "false"   
    IS_SCA_ENABLED= "false"  
    IS_DAST_ENABLED= "false"  
    IS_IMAGE_SCAN_ENABLED= "false"  
    IS_CODE_REVIEW_ENABLED= "false"  
    IS_PEN_TESTING_ENABLED= "false"  
  }

  stages {
    stage('Build') {
      steps {
        sh 'mvn -e clean package -DskipTests'
      }
    }
    stage('SAST - Coverity') {
      steps {
        echo "Stage - Coverity on Polaris"
        sh '''
          IS_SAST_ENABLED=$(jq -r '.security.activities.sast.enabled' result.json)
          echo "IS_SAST_ENABLED = ${IS_SAST_ENABLED}"
          if [ ${IS_SAST_ENABLED} = "true" ]; then
            echo "Running Coverity on Polaris based on IO Precription"
            rm -fr /tmp/polaris
            wget -q ${POLARIS_SERVER_URL}/api/tools/polaris_cli-linux64.zip
            unzip -j polaris_cli-linux64.zip -d /tmp
            /tmp/polaris --persist-config --co project.name="${IO_POC_PROJECT_NAME}" --co capture.build.buildCommands="null" --co capture.build.cleanCommands="null" --co capture.fileSystem="null" --co capture.coverity.autoCapture="enable"  configure
            /tmp/polaris analyze -w
          else
            echo "Skipping Coverity on Polaris based on IO Precription"
          fi
          '''
      }
    }
    stage('SCA - Blackduck') {
      steps {
        echo "Stage - Blackduck"
        sh '''
          IS_SCA_ENABLED=$(jq -r '.security.activities.sca.enabled' result.json)
          echo "IS_SCA_ENABLED = ${IS_SCA_ENABLED}"
          if [ ${IS_SCA_ENABLED} = "true" ]; then
            echo "Running Blackduck based on IO Precription"
            rm -fr /tmp/detect7.sh
            curl -s -L https://detect.synopsys.com/detect7.sh > /tmp/detect7.sh
            bash /tmp/detect7.sh --blackduck.url="${BLACKDUCK_URL}" --blackduck.api.token="${BLACKDUCK_ACCESS_TOKEN}" --detect.project.name="${IO_POC_PROJECT_NAME}" --detect.project.version.name=${IO_POC_PROJECT_VERSION} --blackduck.trust.cert=true
          else
            echo "Skipping Blackduck based on IO Precription"
          fi
          '''
      }
    }
    stage('IO Workflow - CodeDx') {
      steps {
        echo "Preparing to run IO Workflow Engine"
        sh '''
          IS_SAST_ENABLED=$(jq -r '.security.activities.sast.enabled' result.json)
          IS_SCA_ENABLED=$(jq -r '.security.activities.sca.enabled' result.json)
          ./prescription.sh \
          --stage="WORKFLOW" \
          --persona="devsecops" \
          --io.url="${IO_URL}" \
          --io.token="${IO_ACCESS_TOKEN}" \
          --manifest.type="json" \
          --asset.id="insecure-bank" \
          --workflow.url="${WORKFLOW_URL}" \
          --workflow.version="${WORKFLOW_CLIENT_VERSION}" \
          --file.change.threshold="10" \
          --sast.rescan.threshold="20" \
          --sca.rescan.threshold="20" \
          --polaris.project.name="${IO_POC_PROJECT_NAME}" \
          --polaris.url="${POLARIS_SERVER_URL}" \
          --polaris.token="${POLARIS_ACCESS_TOKEN}" \
          --blackduck.project.name="${IO_POC_PROJECT_NAME}:${IO_POC_PROJECT_VERSION}" \
          --blackduck.url="${BLACKDUCK_URL}" \
          --blackduck.api.token="${BLACKDUCK_ACCESS_TOKEN}" \
          --jira.enable="false" \
          --codedx.url="${CODEDX_SERVER_URL}" \
          --codedx.api.key="${CODEDX_ACCESS_TOKEN}" \
          --codedx.project.id="5" \
          --IS_SAST_ENABLED="${IS_SAST_ENABLED}" \
          --IS_SCA_ENABLED="${IS_SCA_ENABLED}" \
          --IS_DAST_ENABLED="${IS_DAST_ENABLED}"
        '''
        echo "Running IO Workflow Engine with CodeDx"
        sh '''
          java -jar WorkflowClient.jar --workflowengine.url="http://52.186.143.163/api/workflowengine/" --io.manifest.path=synopsys-io.json
        '''
      }
    }
    stage('Schedule Manual Activities') {
      steps {
        echo "Check for Scheduling of Manual Actviities"
        echo "Manual Code Review"
        sh '''
          IS_CODE_REVIEW_ENABLED=$(jq -r '.security.activities.sastplusm.enabled' io-presciption.json)
          echo "IS_CODE_REVIEW_ENABLED = ${IS_CODE_REVIEW_ENABLED}"
          if [ ${IS_CODE_REVIEW_ENABLED} = "true" ]; then
            echo "Sending Notification for Manual Code Review based on IO Precription"
            # Put code to send notification here
          else
            echo "Skipping Manual Code Review based on IO Precription"
          fi
          '''
        echo "Manual Penetration Testing"
        sh '''
          IS_PEN_TESTING_ENABLED=$(jq -r '.security.activities.dastplusm.enabled' io-presciption.json)
          echo "IS_PEN_TESTING_ENABLED = ${IS_PEN_TESTING_ENABLED}"
          if [ ${IS_PEN_TESTING_ENABLED} = "true" ]; then
            echo "Sending Notification for Manual Penetration Testing based on IO Precription"
            # Put code to send notification here
          else
            echo "Skipping Manual Penetration Testing based on IO Precription"
          fi
          '''
      }
    }
    stage('Break the Build') {
      steps {
        echo "add Build Breaker parts here"
        sh '''
          echo "Breaker Status - $(jq -r '.breaker.status' wf-output.json)"
          # Put code to break the build here
        '''
      }
    }
    stage('Clean Workspace') {
      steps {
        cleanWs()
      }
    }
  }
}
