def GIT_URL = 'https://github.com/casanovapick/BindingSample.git'
def BRANCH = 'master'
def projectVersion='1.0.0 (build)'
node {
      stage('Preparation') {
         git url: "${GIT_URL}" ,branch: "${BRANCH.replaceAll(".*/","")}"
         sh "git clean -fdx"
         sh "./gradlew clean"
         sh "./gradlew increaseBuildNumber"
      }

      stage('Unit Tests') {
         sh "./gradlew testDebug"
      }

      stage('Static code analysis'){
         def props = readProperties file: 'version.properties'
         def build = props['build']
         def versionName = props['versionName']
         projectVersion = "${versionName}build${build}"
         def workspace = pwd() 
         def SONAR_SETTING = "${workspace}@script/sonar-project.properties"
         sh "${env.SONAR_SCANNER_HOME}/sonar-scanner  -Dproject.settings=${SONAR_SETTING} -Dsonar.projectVersion=${projectVersion}"
      }

      stage('Packaging') {
          try {
            sh "git clone https://github.com/CyberioMor/keystore"
            sh "./gradlew assemble"
         }finally{
            sh "rm -rf keystore"
         }
      }

      stage('Upload Google play'){
         androidApkUpload apkFilesPattern: '**/*-release.apk', googleCredentialsId: 'SampleBinding', trackName: 'beta'
      }

      stage('Upload'){
         sh "zip -rj artifact.zip . -i *.apk"
         nexusArtifactUploader artifacts: [[artifactId: 'sample', classifier: '', file: 'artifact.zip', type: 'zip']], credentialsId: 'cedf432b-2dd2-4e3c-ae6a-11780cf6244d', groupId: 'mobile.android', nexusUrl: 'localhost:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-releases', version: projectVersion
      }

      stage('Update new build number'){
         sh "git commit -m \"${projectVersion}\" version.properties"
         sh "git push origin ${BRANCH.replaceAll(".*/","")}"
      }

}

