pipeline { 
	agent any
	options {
		skipDefaultCheckout(true)
		timeout(time: 1, unit: 'HOURS')
		buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '5', numToKeepStr: '5')
	}
	environment { 
		def jdk=tool name: 'jdk1.8_172'
		def JAVA_HOME="${jdk}"
		def BUILD_NUMBER="${env.BUILD_NUMBER}"
		def WORKSPACE="${env.WORKSPACE}"
		def MAVEN_VERSION=""
		def TAG_NAME="${env.TAG_NAME}"
		def build_timestamp="${new Date().format( 'yyMMdd-HHmm' )}"
	}
	stages {
		stage('Initial Setup') {
			steps {
				script {
					if (isUnix()) { 
						shell="sh"
					} else {
						shell="bat"
					}
					echo "clean workspace\n"
					deleteDir()
				}
			}
		}
		stage('Checkout Repository') {
			steps {
				script {
					echo "checkout repository\n"
					def scmVars=checkout scm
					GIT_BRANCH="${scmVars.GIT_BRANCH}"
					GIT_COMMIT="${scmVars.GIT_COMMIT}"
					GIT_SHORT_COMMIT=GIT_COMMIT.substring(0,7)
					GIT_URL="${scmVars.GIT_URL}"
					echo "**********************"
					echo "GIT_COMMIT=${scmVars.GIT_COMMIT}"
					echo "GIT_BRANCH=${scmVars.GIT_BRANCH}"
					echo "GIT_URL=${scmVars.GIT_URL}"
					echo "GIT_URL=${GIT_URL}/commit/${GIT_SHORT_COMMIT}\n"
					echo "**********************"
					if (GIT_URL.endsWith(".git")) {
						GIT_URL = GIT_URL.substring(0, GIT_URL.length() - 4)
					}
					if ("${TAG_NAME}"=="null") {
						MAVEN_VERSION="${build_timestamp}.${GIT_SHORT_COMMIT}"
					} else { 
						MAVEN_VERSION="${TAG_NAME}"
					}
				}
			}
		}
		stage('Build') {
			steps {
				script {
					echo "MAVEN_VERSION=${MAVEN_VERSION}"
					echo "Interpolating NPM configuration"
					withCredentials([usernamePassword(credentialsId: 'NexusLogin', passwordVariable: 'sdpass', usernameVariable: 'sduser')]) {
						configFileProvider([configFile(fileId: 'npmrc-aem', replaceTokens: true, targetLocation: '.')]) {}
						withMaven(jdk: 'jdk1.8_172', maven: 'mvn363', mavenSettingsConfig: 'sd_maven_settings_nexus') {
							"${shell}" "mvn -U clean package -Prelease -DskipTests"
						}
					}
				}
			}
		}
		stage('CleanUp') {
			steps {
				script {
					echo "Cleaning UP"
					"${shell}" "ls -lrth ${WORKSPACE}"
					"${shell}" "rm -f ${WORKSPACE}/.npmrc"
					"${shell}" "rm -f ${WORKSPACE}/.npmrc_bak"
				}
			}
		}
	}
}