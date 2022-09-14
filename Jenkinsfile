pipeline {
    agent any

	tools {
	    jdk 'JDK 11'
		maven 'Maven_3.8.5'
	}
	parameters { string(name: 'JAIN_SLEE_SIP_MAJOR_VERSION', defaultValue: '7.2.0', description: 'The major version for JAIN SLEE SIP version') }

    environment{
        SIP_BUILD_VERSION = "${params.JAIN_SLEE_SIP_MAJOR_VERSION}-${BUILD_NUMBER}"
    }

	stages{
		stage("Build ") {
			steps {
			    sh 'java -version'
				script {
				    SIP_BUILD_VERSION = "${params.RESTCOMM_SLEE_JDBC_VERSION}-${BUILD_NUMBER}"
                    currentBuild.displayName = "#${SIP_BUILD_VERSION}"
                    currentBuild.description = "JAIN SLEE SIP (${env.BRANCH_NAME})"
                }
				echo "Building JAIN SLEE SIP version (#${SIP_BUILD_VERSION})"
                sh "mvn versions:set -DgenerateBackupPoms=false -DnewVersion=${SIP_BUILD_VERSION} clean install -DskipTests"
				sh 'mvn clean install -DskipTests'
                echo "Maven build completed."
            }
        }

		stage("Release") {
			steps {
				echo "Building a wildfly release version of #${SIP_BUILD_VERSION}"
                sh 'find . -type d -name \\"target\\" -exec r -r {} +'
                sh "mvn versions:set -DnewVersion=${SIP_BUILD_VERSION} clean install -Prelease -Drelease.dir=../../../${SIP_BUILD_VERSION} -Dmaven.test.skip=true"
                echo "Building wildfly version completed."
            }
        }

        stage("Zip Resources") {
            steps{
                dir("${SIP_BUILD_VERSION}/resources"){
                    sh "zip -r sip11.zip sip11"
                    echo 'Deleting sub folders'
                    sh 'rm -rf sip11'
                }
            }
        }

        stage("Zip Enablers") {
            steps{
                echo "Zipping enablers"
                dir("${SIP_BUILD_VERSION}/enablers"){
                    sh 'zip -r sip-publication-client.zip sip-publication-client'
                    sh 'zip -r sip-subscription-client.zip sip-subscription-client'
                    sh 'rm -rf sip-publication-client sip-subscription-client'
                }
            }
		}

        stage("Zip Examples") {
            steps{
                echo "Zipping examples"
                dir("${SIP_BUILD_VERSION}/examples"){
                    sh 'zip -r sip-wake-up.zip sip-wake-up'
                    sh 'zip -r sip-uas.zip sip-uas'
                    sh 'zip -r sip-jdbc-registrar.zip sip-jdbc-registrar'
                    sh 'zip -r sip-b2bua.zip sip-b2bua'
                    sh 'rm -rf sip-wake-up sip-uas sip-b2bua sip-jdbc-registrar'
                }
            }
		}

		stage('Save Artifacts') {
            steps {
                echo "Archiving JAIN SLEE SIP version ${SIP_BUILD_VERSION}"
                archiveArtifacts artifacts: "${SIP_BUILD_VERSION}/", followSymlinks: false, onlyIfSuccessful: true
            }
        }

        stage('Push to Repo') {
            when{anyOf {branch 'master'; branch 'release'}}
			steps {
				sh "mkdir -p /var/www/html/NAIKERI/jain-slee.sip/${SIP_BUILD_VERSION}/"
                sh "cp ${SIP_BUILD_VERSION}/ /var/www/html/NAIKERI/jain-slee.sip/${SIP_BUILD_VERSION}/"
                /*sshagent (credentials: ['4e708f2a-8b37-414f-a6d4-787690b87738']) {
                    sh 'scp -r ${SIP_BUILD_VERSION}/ root@127.0.0.1:/var/www/html/NAIKERI/jain-slee.sip/${SIP_BUILD_VERSION}/'
                }*/

				sh "rm -rf ${SIP_BUILD_VERSION}"
			}
		}
	}

	post {
		success {
			echo "JAIN-SLEE SIP successfully built"
		}
    failure {
      echo "Failed to build JAIN-SLEE SIP"
    }
		always {
			echo "This will be called always. After testing do clean up"
		}
	}
}