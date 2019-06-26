node {
  stage('SCM Checkout'){
     	git 'https://github.com/jpbejedor/CaseStudy1.git'
	  
    slackSend baseUrl: 'https://hooks.slack.com/services/', 
    channel: '#devops', 
    color: 'good', 
    iconEmoji: '', 
    message: 'SCM Now Checked Out!', 
    teamDomain: 'DEVOPS', tokenCredentialId: 'Slack_Token', username: 'jp.bedejor@sprint.com'
  
  }
    
  stage('BUILD'){
  def mvnHome = tool name: 'maven3.6.1', type: 'maven'
    	sh "${mvnHome}/bin/mvn clean package"
   }
	
  stage('TEST'){
  def scannerHome = tool 'SonarQubeScanner'
	withSonarQubeEnv('SonarQube') {
      	sh "${scannerHome}/bin/sonar-scanner"	
    }
  }
	
   stage("Quality Gate Status Check"){
          timeout(time: 1, unit: 'HOURS') {
              def qg = waitForQualityGate()
              if (qg.status != 'OK') {
                   //slackSend baseUrl: 'https://hooks.slack.com/services/', 
                   // channel: '#devops', 
                    //color: 'danger', 
		      sh "echo 'auto deployment every minute'"
                    //iconEmoji: '', 
                    //message: 'SonarQube Analysis Failed!', 
                   // teamDomain: 'DEVOPS', tokenCredentialId: 'Slack_Token', username: ''
                  error "Pipeline aborted due to quality gate failure: ${qg.status}"
              }
          }
      }
	
  stage ('DEPLOY'){
	  sh "echo 'Deploying to Tomcat'"
	  sh "whoami"
	  sshagent(credentials: [''], ignoreMissing: true){
  def source = '/Users/Shared/Jenkins/Home/workspace/CaseStudy1Pipeline/target/*.war'
  def target = '/Library/Tomcat/webapps/'
	  sh "cp $source $target"
	  }
  } 

  stage ('Post Deploy Test'){
      sleep 10  
	  
     //def Response = sh "curl -LI http://localhost:8082/myweb-0.0.1-SNAPSHOT -o /dev/null -w '%{http_code}\n' -s"
       def response = sh (script: "curl -LI http://localhost:8082/myweb-0.0.1-SNAPSHOT -o /dev/null -w '%{http_code}\n' -s", returnStdout: true).trim()
       sh "echo 'Response is $response' "
	  if (response != '200'){
	   error "Pipeline aborted due to Site failure: $response"
	  }else
	  sh "echo 'Successfully Accessed the site with return code $response'"
  }
	
	
  stage ('UPLOAD Artifactory'){
  def server = Artifactory.server('MyArtifactory')	
  def rtMaven = Artifactory.newMavenBuild()
  	rtMaven.resolver releaseRepo: 'maven', snapshotRepo: 'maven'
  	rtMaven.deployer server: server, releaseRepo: 'lib-release-local', snapshotRepo: 'lib-snapshot-local'
  	rtMaven.tool = 'maven3.6.1'
  def buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install'
  	server.publishBuildInfo buildInfo  
  }	
	
   stage ('Slack Notification'){ 
    slackSend baseUrl: 'https://hooks.slack.com/services/', 
    channel: '#devops', 
    color: 'good', 
    iconEmoji: '', 
    message: 'Your Build is now Deployed!', 
    teamDomain: 'DEVOPS', tokenCredentialId: 'Slack_Token', username: 'jp.bedejor@sprint.com'
  
   }
	
	stage('Email Notification'){
       mail bcc: '', body: '''Hi Team! 
Your Build is now Deployed!

Thanks.

Kind Regards,

JP Bejedor''', cc: 'jp.bedejor@sprint.com', from: '', replyTo: '', subject: 'DevOps Jenkins Job Notif', to: 'igsp-eai-prod@sprint.com'
	
	}		
}

