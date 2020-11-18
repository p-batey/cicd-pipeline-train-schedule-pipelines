pipeline {
agent any
stages {
	stage('Build') {
		steps {
			echo 'Running build automation'
			sh './gradlew build --no-daemon'
			archiveArtifacts artifacts: 'dist/trainSchedule.zip'
		}
	}
	stage('DeployToStaging') {
		when {
			branch 'master'		--> conditional when, all need to be true
		}
		steps {
			withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
				sshPublisher{
					failsOnError: true,
					continueOnError: false,
					publishers: [
						sshPublishDesc(
							configName: 'staging',					--> staging server from the credentials store (same name)
							sshCredentials: [
								username: "$USERNAME",
								encryptedPassphrase: "$USERPASS"
							],
							transfers: [
								sshTransfer(
									sourceFiles: 'dist/trainSchedule.zip',	--> The build stage generates this
									removePrefix: 'dist/',					--> only move the zip not the dist folder
									remoteDirectory: 'tmp',
									execCommand: 'sudo /usr/bin/systemctl stop train-schedule && re -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/ssytemctl start train-schedule'
								)
							]
						)
					]
				)
			}
		}
	}
}
stage('DeployToProduction') {
		when {
			branch 'master'
		}
		steps {
			input 'Does the staging environment look OK?'
			milestone(1)	--> prevents an older build from being deployed if whilst pasued another build is started
			withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariabe: 'USERPASS'
				sshPublisher{
					failsOnError: true,
					continueOnError: false,
					publishers: [
						sshPublishDesc(
							configName: 'production',
							sshCredentials: [
								username: "$USERNAME",
								encryptedPassphrase: "$USERPASS"
							],								],
							transfers: [
								sshTransfer(
									sourceFiles: 'dist/trainSchedule.zip',	--> The build stage generates this
									removePrefix: 'dist/',					--> only move the zip not the dist folder
									remoteDirectory: 'tmp',
									execCommand: 'sudo /usr/bin/systemctl stop train-schedule && re -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/ssytemctl start train-schedule'
									)
								]
							)
						]
					)
				}
			}
		}
	}
}
