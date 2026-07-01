pipeline {
	agent any
	tools {
        jdk 'jdk-11'
        maven 'maven-3.9.12'
    }
	options {
    	buildDiscarder(logRotator(daysToKeepStr: '10', numToKeepStr: '10'))
  	}
	parameters {
		string(name: 'JSLEE_SIP_VERSION', defaultValue: '7.2.0', description: 'The major version for JAIN SLEE SIP')
	}
	environment {
        SIP_BUILD_VERSION = "${params.JSLEE_SIP_VERSION}-${BUILD_NUMBER}"
    }
	stages {
		stage("Build") {
			steps {
				script {
                    SIP_BUILD_VERSION = "${params.JSLEE_SIP_VERSION}-${BUILD_NUMBER}"
                    currentBuild.displayName = "#${SIP_BUILD_VERSION}"
                    currentBuild.description = "JAIN SLEE SIP (${env.BRANCH_NAME})"
                }
                sh "mvn clean install -Dmaven.test.skip=true"
			}
		}
		stage('Set Version') {
			steps {
				sh "mvn versions:set -DgenerateBackupPoms=false -DnewVersion=${SIP_BUILD_VERSION}"
                sh "mvn versions:commit"
			}
		}
		stage("Release") {
			steps {
                sh "mvn clean install -Prelease -Drelease.dir=../../../${SIP_BUILD_VERSION} -Dmaven.test.skip=true"
			}
		}
		stage('Zip Resources') {
			steps {
				dir("${SIP_BUILD_VERSION}/resources") {
					sh "zip -r sip.zip sip11"
					sh 'rm -rf sip11'
				}
			}
		}
		stage('Save Artifacts') {
            steps {
                archiveArtifacts artifacts: "${SIP_BUILD_VERSION}/", followSymlinks: false, onlyIfSuccessful: true
            }
		}
        stage('Push to Repo') {
            when{anyOf {branch 'master'; branch 'release'}}
			steps {
                sh "mkdir -p /var/www/html/NAIKERI/jain-slee.sip/${SIP_BUILD_VERSION}/"
                sh "cp -r ${SIP_BUILD_VERSION}/ /var/www/html/NAIKERI/jain-slee.sip/${SIP_BUILD_VERSION}/"
				sh "rm -rf ${SIP_BUILD_VERSION}"
			}
		}
    }
	post {
		success { echo "JAIN-SLEE SIP successfully built" }
		failure { echo "Building JAIN-SLEE SIP failed" }
	}
}
