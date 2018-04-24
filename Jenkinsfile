def GIT_URL = 'https://github.com/casanovapick/BindingSample.git'
def BRANCH = 'master'
def projectVersion='1.0.0 (build)'
node {
   stage('Preparation') {
      git url: "${GIT_URL}" ,branch: "${BRANCH.replaceAll(".*/","")}"
      sh "git clean -fdx"
      sh "./gradlew clean"
      sh "./gredlew increaseBuildNumber"
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
      sh "/Users/wannnasit.chaiphinan/Library/SonarScanner/bin/sonar-scanner  -Dproject.settings=${SONAR_SETTING} -Dsonar.projectVersion=${projectVersion}"
   }

   stage('Packaging') {
      sh "./gradlew assemble"
   }

   stage('Upload'){
      sh "zip -r artifact.zip . -i *.apk"
      nexusArtifactUploader artifacts: [[artifactId: 'sample', classifier: '', file: 'artifact.zip', type: 'zip']], credentialsId: 'cedf432b-2dd2-4e3c-ae6a-11780cf6244d', groupId: 'mobile.android', nexusUrl: 'localhost:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-releases', version: projectVersion
   }

   stage('Update new build number'){

      sh "git commit -m \"${projectVersion}\" version.properties"
      sh "git push"
   }
}