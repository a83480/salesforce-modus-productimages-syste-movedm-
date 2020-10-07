node {
	// this jenkinsfile is only for mule 4 app
	// for mule 3 app please use its corresponding jenkinsfile
	def jobName = env.JOB_NAME
	def projectName = jobName.split('/')[0]
	def mvn_version = 'M3'
	// for on going we only support deploying to one server on one platform
	def platform = 'azcloud'	// platform i.e. onprem or azcloud 
	def server = 'it2'		// server runtime prefix i.e. it1, it2, ...
	def version
	def group
	def deployCode
	def fileSizeReturn
	def int fileSize

	//def webhook = 'https://outlook.office.com/webhook/67a43724-8724-4996-85fb-29e92ee8e159@5b44bfa6-47bc-4638-8409-a21d403f820a/JenkinsCI/14f9527561f040c9b2101440c94f59bb/7207ec74-f7cb-4d0e-be0f-0f73058b5c79'
	def webhook=env.TEAM_WEBHOOK
		
	try {
	
		//hipchatSend(color: "YELLOW", 
		//	message: "STARTED Job : ${env.JOB_NAME} [build# ${env.BUILD_NUMBER}]")
		office365ConnectorSend message: "STARTED Job : ${env.JOB_NAME} [build# ${env.BUILD_NUMBER}]", 
			status:"Started", webhookUrl:"${webhook}", color: "ffff00"
		
		stage ('Preparation') {
			println "Build: " + projectName + " for branch " + env.BRANCH_NAME
			git(
				url: "https://github.com/andersencorp/${projectName}.git",
				credentialsId: 'EAIGitHub',
				branch: env.BRANCH_NAME
			)		
			sh 'git config credential.helper store' // change from cache to store
			sh 'git config push.default simple'
			sh 'git clean -f'
			sh 'git reset --hard'
			sh 'git checkout .'
		}
		stage ('Build') {
			if (fileExists("${projectName}")) {
				if (env.BRANCH_NAME=='integrations4') {
					echo 'build integrations branch'
					withEnv( ["PATH+MAVEN=${tool mvn_version}/bin"] ) {
						echo "${tool mvn_version}"
						def pom = readMavenPom file: "${projectName}/pom.xml"
						def versionList = pom.version.replace("-SNAPSHOT", "").tokenize(".")
						version = "${versionList[0]}.${versionList[1]}.${versionList[2]}"
						group = pom.groupId.replace(".", "/")
						echo "${group}"
						dir ("${projectName}") {
							sh 'mvn clean package -Dmule.env=dev -Dmule.hostname=bpmnplcid01.andersencorp.com'
							dir ("target") {
								deployCode = sh(returnStdout:true, script: "/mule/scripts/deploy4.sh push Development ${projectName} ${server} ${platform} ${version}")
								if (deployCode.contains('"message": "Deployment Completed Successfully"')) {
									println('app deploy success')
								} else if (deployCode.contains('"message": "Check Application Status in Anypoint Runtime Manager"')) {
									println('please check application status to make sure it is deployed and started successfully')
									//hipchatSend(color: "PURPLE", 
									//	message: "WARNING: Please check your application in Runtime Manager to make sure it is deployed and started successfully")									
									office365ConnectorSend message: "WARNING: Please check your application in Runtime Manager to make sure it is deployed and started successfully", 
										status:"Warning", webhookUrl:"${webhook}", color: "800080"
								} else {
									println('app deploy fail')
									println(deployCode)
									error('app deploy fail')
								}
								fileSizeReturn = sh(returnStdout:true, script: "/mule/scripts/appFileSize.sh Development ${projectName} ${server} ${platform} ${version}").trim()
								fileSize = Integer.parseInt(fileSizeReturn)
								println('fileSize: ')
								println(fileSize)
								if (fileSize > 30) {
									//hipchatSend(color: "PURPLE", 
									//	message: "WARNING: Application file for ${projectName} ${server} ${platform} ${version} is greater than 30M")
									office365ConnectorSend message: "WARNING: Application file for ${projectName} ${server} ${platform} ${version} is greater than 30M", 
										status:"Warning", webhookUrl:"${webhook}", color: "800080"
								}
							}
						}
					}
	
				} else if (env.BRANCH_NAME=='master4') {
					echo 'build master branch'
					withEnv( ["PATH+MAVEN=${tool mvn_version}/bin"] ) {
						def pom = readMavenPom file: "${projectName}/pom.xml"
						def versionList = pom.version.replace("-SNAPSHOT", "").tokenize(".")
						def newRelease = Eval.me(versionList[2])+1
						echo "${newRelease}"
						version = "${versionList[0]}.${versionList[1]}.${versionList[2]}"
						def newversion="${versionList[0]}.${versionList[1]}.${newRelease}-SNAPSHOT"
						echo "release version: ${version}"
						echo "new version: ${newversion}"
						group = pom.groupId.replace(".", "/")
						echo "${group}"
						dir ("${projectName}") {
							echo "release version: ${version}"
							sh "mvn clean versions:set -DnewVersion=${version}"
							sh 'mvn clean deploy -Dmule.env=stg -Dmule.hostname=bpmnplcid01.andersencorp.com'
							dir ('target') {
								deployCode = sh(returnStdout:true, script: "/mule/scripts/deploy4.sh push Stage ${projectName} ${server} ${platform} ${version}")
								if (deployCode.contains('"message": "Deployment Completed Successfully"')) {
									println('app deploy success')
								} else if (deployCode.contains('"message": "Check Application Status in Anypoint Runtime Manager"')) {
									println('please check application status to make sure it is deployed and started successfully')
									//hipchatSend(color: "PURPLE", 
									//	message: "WARNING: Please check your application in Runtime Manager to make sure it is deployed and started successfully")									
									office365ConnectorSend message: "WARNING: Please check your application in Runtime Manager to make sure it is deployed and started successfully", 
										status:"Warning", webhookUrl:"${webhook}", color: "800080"
								} else {
									println('app deploy fail')
									println(deployCode)
									error('app deploy fail')
								}
								fileSizeReturn = sh(returnStdout:true, script: "/mule/scripts/appFileSize.sh Stage ${projectName} ${server} ${platform} ${version}").trim()
								fileSize = Integer.parseInt(fileSizeReturn)
								println('fileSize: ')
								println(fileSize)
								if (fileSize > 30) {
									//hipchatSend(color: "PURPLE", 
									//	message: "WARNING: Application file for ${projectName} ${server} ${platform} ${version} is greater than 30M")
									office365ConnectorSend message: "WARNING: Application file for ${projectName} ${server} ${platform} ${version} is greater than 30M", 
										status:"Warning", webhookUrl:"${webhook}", color: "800080"
								}

								sh "cp ${projectName}-${version}-mule-application.jar /jenkins/release"
							}
						}
						sh "git tag -f ${projectName}-${version} -m JenkinsRelease-${version}"
						sh 'git push -u origin master4 --tag'
						sh 'git checkout .'
						sh 'git checkout integrations4'
						sh 'git clean -f'
						sh 'git reset --hard'
						sh 'git checkout .'
						sh 'git pull origin integrations4'
						dir ("${projectName}") {
							sh "mvn clean versions:set -DnewVersion=${newversion}"
							sh 'git add -- pom.xml'
						}
						sh "git commit -m jenkinsChangingVersion-${newversion}"
						sh 'git push -u origin integrations4'		
					}
	
				}
			} else {
				echo "${projectName} does not exist in repository"
				echo "No build is done"
			}				
		}
		//hipchatSend(color: "GREEN", 
		//	message: "<img src='https://dujrsrsgsd3nh.cloudfront.net/img/emoticons/successful-1414026553.png'/> SUCCESSFUL Job : ${env.JOB_NAME} [build# ${env.BUILD_NUMBER}] (<a href='${env.BUILD_URL}/console'>Details</a>)")
		office365ConnectorSend message: "SUCCESSFUL Job : ${env.JOB_NAME} [build# ${env.BUILD_NUMBER}] (<a href='${env.BUILD_URL}/console'>Details</a>)", 
			status:"Success", webhookUrl:"${webhook}", color: "00ff00"
		
	} catch (e) {
		//hipchatSend(color: "RED", 
		//	message: "<img src='https://dujrsrsgsd3nh.cloudfront.net/img/emoticons/failed-1414026532.png'/> FAILED Job : ${env.JOB_NAME} [build# ${env.BUILD_NUMBER}] (<a href='${env.BUILD_URL}/console'>Details</a>)")
		office365ConnectorSend message: "FAILED Job : ${env.JOB_NAME} [build# ${env.BUILD_NUMBER}] (<a href='${env.BUILD_URL}/console'>Details</a>)", 
			status:"Failure", webhookUrl:"${webhook}", color: "ff0000"
		if (env.BRANCH_NAME=='master4') {
			withCredentials([usernamePassword(credentialsId: 'nexus', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
				sh "curl --request DELETE --user '${USERNAME}:${PASSWORD}' http://nexusrepo.andersencorp.com:8081/nexus/service/local/repositories/integration-releases/content/${group}/${projectName}/${version}"
			}
			//hipchatSend(color: "RED", 
			//	message: "<img src='https://dujrsrsgsd3nh.cloudfront.net/img/emoticons/failed-1414026532.png'/> FAILED Job : ${env.JOB_NAME} [build# ${env.BUILD_NUMBER}] artifact ${group} ${projectName} ${version} is deleted from nexus")
			office365ConnectorSend message: "FAILED Job : ${env.JOB_NAME} [build# ${env.BUILD_NUMBER}] artifact ${group} ${projectName} ${version} is deleted from nexus", 
				status:"Failure", webhookUrl:"${webhook}", color: "ff0000"
		}
		throw e;
	}
}
