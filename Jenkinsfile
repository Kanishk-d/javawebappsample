import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles) {
    if (p['publishMethod'] == 'FTP') {
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
    }
  }
}

node {
  withEnv([
    'AZURE_SUBSCRIPTION_ID=a778bbf9-56d1-4e99-b19a-0ecd2b322cf8',
    'AZURE_TENANT_ID=a9e61550-cf79-4349-83d3-dec5440cc70c'
  ]) {

    stage('init') {
      checkout scm
    }

    stage('build') {
      sh 'mvn clean package'
    }

    stage('deploy') {
      def resourceGroup = 'jenkins-get-started-rg'
      def webAppName = 'jenkins-demo-app123'

      withCredentials([usernamePassword(
          credentialsId: 'AzureServicePrincipal',
          usernameVariable: 'AZURE_CLIENT_ID',
          passwordVariable: 'AZURE_CLIENT_SECRET'
      )]) {
        sh '''
          az login --service-principal \
            --username $AZURE_CLIENT_ID \
            --password $AZURE_CLIENT_SECRET \
            --tenant $AZURE_TENANT_ID

          az account set --subscription $AZURE_SUBSCRIPTION_ID

          az webapp deploy \
            --resource-group jenkins-get-started-rg \
            --name jenkins-demo-app123 \
            --src-path target/calculator-1.0.war \
            --type war
        '''
      }

      // Optional: If you want FTP upload too
      // Uncomment below only if FTP upload is needed
      /*
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile(pubProfilesJson)
      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      */

      sh 'az logout'
    
