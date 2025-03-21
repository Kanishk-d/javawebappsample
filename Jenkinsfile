import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=a778bbf9-56d1-4e99-b19a-0ecd2b322cf8',
        'AZURE_TENANT_ID=a9e61550-cf79-4349-83d3-dec5440cc70c']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'jenkins-get-started-rg'
      def webAppName = '<app_name>'
      // login Azure
withCredentials([usernamePassword(
    credentialsId: 'AzureServicePrincipal',
    usernameVariable: 'd9fd06c9-2139-4283-a06a-f154db583b1e',
    passwordVariable: 'xTQ8Q~rXqv120Pd_H6qM9ZdjO_Wr7axXNDbETc_r'
)]) {
    sh '''
        az login --service-principal \
                 --username $AZURE_CLIENT_ID \
                 --password $AZURE_CLIENT_SECRET \
                 --tenant a9e61550-cf79-4349-83d3-dec5440cc70c

        az account set --subscription a778bbf9-56d1-4e99-b19a-0ecd2b322cf8
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
