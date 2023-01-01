node {
    
    //properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: '5')), [$class: 'JobLocalConfiguration', changeReasonComment: ''], pipelineTriggers([cron('* * * * *')])])
    properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: '5')), [$class: 'JobLocalConfiguration', changeReasonComment: ''], pipelineTriggers([githubPush()])])
    
    echo "Build display name is :  ${env.BUILD_DISPLAY_NAME}"
    buildName "Dev Branch : ${env.BUILD_NUMBER}" 
    
    def mavenHome = tool name: "maven3.8.6"
    //echo "maven version is -------> : "${maven}
    
	
	try {
		    stage('checkOutCode'){
			git branch: 'development', credentialsId: '70b2989d-2fd0-4d67-b14a-01d0bb721ec9', url: 'https://github.com/devopstrainingnandu/maven-web-application.git'
		    }

		    stage('Build'){
			sh "${mavenHome}/bin/mvn clean package"
		    }

		    stage('sqReport'){
			sh "${mavenHome}/bin/mvn sonar:sonar"
		    }

		    stage('artifactUpload'){
			sh "${mavenHome}/bin/mvn deploy"
		    }

		    stage('deployToTomcatServer'){
				sshagent(['f3b7c344-307f-4350-b4c6-abfabc2e01b8']) {
				    sh "scp -o StrictHostKeyChecking=no target/maven-web-application.war ec2-user@43.205.231.102:/opt/apache-tomcat-9.0.70/webapps/"
				}
		    }

		    stage('SlackNotification'){
			slackSend channel: '#pipeline_scriptedway', tokenCredentialId: '914dde1e-72af-414c-ad91-490f07c9d828'
		    }
	}
	catch(e){
		currentBuild.result = "FAILED"
	
	}
	finally{
	        sendSlackNotifications(currentBuild.result)
	}
    
}



//Slack Send Notifications

def sendSlackNotifications(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESS'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    colorName = 'ORANGE'
    colorCode = '#FFA500'
  } else if (buildStatus == 'SUCCESS') {
    colorName = 'GREEN'
    colorCode = '#00FF00'
  } else {
    colorName = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary, channel: '#icicibank')
}


