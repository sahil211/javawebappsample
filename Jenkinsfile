import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=a98a10ce-e35d-43b1-bac5-d63d198f6ffd',
        'AZURE_TENANT_ID=0adb040b-ca22-4ca6-9447-ab7b049a22ff']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'QuickstartJenkins-rg'
      def webAppName = 'myjenkins-app'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: 'EAmw.XgGn5eO--00N8f63GCkPtbkbb_R79', usernameVariable: 'f1c18a96-1728-4549-8bb8-cb45c4a64559')]) {
       sh '''
          az login --service-principal -u f1c18a96-1728-4549-8bb8-cb45c4a64559 -p EAmw.XgGn5eO--00N8f63GCkPtbkbb_R79 -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
      sh 'az logout'
    }
  }
}
