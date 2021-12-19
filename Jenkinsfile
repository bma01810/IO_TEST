pipeline {
  agent any

  environment {
    IO_POC_PROJECT_NAME = 'bma_IO-POC-insecure-bank'
    IO_POC_PROJECT_VERSION = "1.0"
    POLARIS_ACCESS_TOKEN = credentials('polaris-token')
    BLACKDUCK_ACCESS_TOKEN = credentials('BlackDuck-AuthToken')
    IS_SAST_ENABLED=$(jq -r '.security.activities.sast.enabled' result.json)    
    IS_SCA_ENABLED=$(jq -r '.security.activities.sca.enabled' result.json)
    IS_DAST_ENABLED=$(jq -r '.security.activities.dast.enabled' result.json)
    IS_IMAGE_SCAN_ENABLED=$(jq -r '.security.activities.imageScan.enabled' result.json)
    IS_CODE_REVIEW_ENABLED=$(jq -r '.security.activities.sastplusm.enabled' result.json)
    IS_PEN_TESTING_ENABLED=$(jq -r '.security.activities.dastplusm.enabled' result.json)
  }

  stages {
    stage('Build') {
      steps {
        sh 'mvn -e clean package -DskipTests'
      }
    }
    stage('SAST - Coverity') {
      steps {
        echo "Running Coverity on Polaris"
        sh '''
          rm -fr /tmp/polaris
          wget -q ${POLARIS_SERVER_URL}/api/tools/polaris_cli-linux64.zip
          unzip -j polaris_cli-linux64.zip -d /tmp
          /tmp/polaris --persist-config --co project.name="${IO_POC_PROJECT_NAME}" --co project.branch="main" --co capture.build.buildCommands="null" --co capture.build.cleanCommands="null" --co capture.fileSystem="null" --co capture.coverity.autoCapture="enable"  configure
          /tmp/polaris analyze -w
        '''
      }
    }
    stage('SCA - Blackduck') {
      steps {
        echo "Running Blackduck"
        sh '''
          rm -fr /tmp/detect7.sh
          curl -s -L https://detect.synopsys.com/detect7.sh > /tmp/detect7.sh
          bash /tmp/detect7.sh --blackduck.url="${BLACKDUCK_URL}" --blackduck.api.token="${BLACKDUCK_ACCESS_TOKEN}" --detect.project.name="${IO_POC_PROJECT_NAME}" --detect.project.version.name=${IO_POC_PROJECT_VERSION} --blackduck.trust.cert=true
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
