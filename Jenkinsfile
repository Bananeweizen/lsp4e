pipeline {
	options {
		timeout(time: 40, unit: 'MINUTES')
		buildDiscarder(logRotator(numToKeepStr: '10'))
	}
	agent {
		label 'centos-latest'
	}
	tools {
		maven 'apache-maven-latest'
		jdk 'openjdk-jdk17-latest'
	}
	stages {
		stage('Build') {
			steps {
				withMaven(maven:'apache-maven-latest', mavenLocalRepo: '$WORKSPACE/.m2/repository') {
				wrap([$class: 'Xvnc', useXauthority: true]) {
					sh '''
						mvn clean verify \
							org.eclipse.dash:license-tool-plugin:license-check \
							-B -Psign \
							-Dmaven.test.failure.ignore=true \
							-Ddash.fail=false \
							-Dsurefire.rerunFailingTestsCount=3
					'''
				}}
			}
			post {
				always {
					archiveArtifacts artifacts: 'repository/target/repository/**/*,repository/target/*.zip,*/target/work/data/.metadata/.log'
					junit '*/target/surefire-reports/TEST-*.xml'
				}
			}
		}
		stage('Deploy Snapshot') {
			when {
				branch 'master'
			}
			steps {
				sshagent (['projects-storage.eclipse.org-bot-ssh']) {
					sh '''
						DOWNLOAD_AREA=/home/data/httpd/download.eclipse.org/lsp4e/snapshots/
						echo DOWNLOAD_AREA=$DOWNLOAD_AREA
						ssh genie.lsp4e@projects-storage.eclipse.org "\
							rm -rf ${DOWNLOAD_AREA}/* && \
							mkdir -p ${DOWNLOAD_AREA}"
						scp -r repository/target/repository/* genie.lsp4e@projects-storage.eclipse.org:${DOWNLOAD_AREA}
					'''
				}
			}
		}
	}
}
