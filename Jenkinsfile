pipeline {
	options {
		buildDiscarder(logRotator(numToKeepStr: '10'))
		disableConcurrentBuilds()
		ansiColor('xterm')
	}
	agent {
		label 'dpkg'
	}
	stages {
		stage('Prepare') {
			steps {
				sh 'rm -rf sign'
				sh 'mkdir -p sign'
				dir(path: 'sign') {
					git(url: "${RPM_SIGN_GIT_REPO}", credentialsId: "${RPM_SIGN_CREDENTIALS_ID}")
					sh 'chmod 600 .gnupg/*'
				}
			}
		}
		stage('Build') {
			steps {
				sh './build rust'
			}
		}
		stage('Publish') {
			steps {
				script {
					def Dpkg = readJSON file: 'dpkg.json'
					archiveArtifacts(artifacts: "${Dpkg.PACKAGE}*.*", caseSensitive: true, onlyIfSuccessful: true)
					dpkgGithubRelease('debian10')
				}
			}
		}
	}
	post {
		success {
			juxtapose event: 'success'
			sh 'figlet "SUCCESS"'
		}
		failure {
			juxtapose event: 'failure'
			sh 'figlet "FAILURE"'
		}
	}
}
