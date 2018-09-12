throttle(['external_grid']) {
	node ('ctm-bsd') {
				def isBuildFailure = false
				def repo
				def gridURL = 'http://SAx7nLkcZ0JCWZj6VHa8RTKX9Umh1SIK:i6HwHSzxJ8ZvY8sJFtInciab3niV5Rve@OD.gridlastic.com:80/wd/hub'
				def application = 'bsd'
				def PARAMS
				def emailList = 'ecommateam@officedepot.com'
				def constants
				def gitId
				def svnId
				def testngFilesList
				def testdataFilesList
				def bsdTestUrl
				def packageName = 'SpecialProjects'
				def mavenContainerVersion
				def dataProviderFileList
				def testngFilePath = 'resources/testng/'
				def testngFailedXml
				def testName
				stage("Read Constants and Shared Library") {
					script {
						constants = evaluate readTrusted('jenkins_pipeline/pipeline_constants.groovy')
						sharedLib = evaluate readTrusted('jenkins_pipeline/common_lib.groovy')
						gitId = "${constants.gitId}"
						testngFilesList = "${constants.testngFilesList}"
						testdataFilesList = "${constants.testdataFilesList}"
						svnId = "${constants.svnId}"
						bsdTestUrl = "${constants.bsdTestUrl}"
						mavenContainerVersion = "${constants.mavenContainerVersion}"
					}
				}
				stage("Get Package Names from SCM") {
					container('ctmalpine') {
						script {
							sh "apk --no-cache add apache2 apache2-utils apache2-webdav mod_dav_svn subversion"
							withCredentials([usernamePassword(credentialsId: "${svnId}", passwordVariable: 'PASS', usernameVariable: 'USER')]) {
								SVN_PACKAGES = sh (
									script: "svn list https://ecomsvn.officedepot.com/repositories/ECOM/trunk/Automation/ClearTestMax/BSD/src/com/officedepot/SpecialProjects/ --username ${USER} --password ${PASS}",
									returnStdout: true
									).trim()
									def testNamesList = SVN_PACKAGES.split("[\\r\\n/]+").collect{it}
									def finalList = []
									def packageNamesWithoutExtension = testNamesList.each { fileName ->
										if(fileName.contains(".java")) {
											def cleanedUpFileName = fileName.substring(0, fileName.lastIndexOf("."))
											if(!(cleanedUpFileName.endsWith("Var")) && !(cleanedUpFileName.endsWith("var"))) {
												finalList.add(cleanedUpFileName)
											}
										}
									}
									env.PACKAGES = finalList.join("\n")
								}
							}
						}
					}
					stage('Get Data Provider Files List') {
						container('ctmalpine') {
							script {
								sh "apk --no-cache add apache2 apache2-utils apache2-webdav mod_dav_svn subversion"
								withCredentials([usernamePassword(credentialsId: "${svnId}", passwordVariable: 'PASS', usernameVariable: 'USER')]) {
									SVN_PACKAGES = sh (
										script: "svn list https://ecomsvn.officedepot.com/repositories/ECOM/trunk/Automation/ClearTestMax/WWW/resources/TestData/ --username ${USER} --password ${PASS}",
										returnStdout: true
										).trim()
										def testNamesList = SVN_PACKAGES.split("[\\r\\n/]+").collect{it}
										def finalList = []
										def packageNamesWithoutExtension = testNamesList.each { fileName ->
											if(fileName.contains(".xml") || fileName.contains(".xls") || fileName.contains(".xlsx")) {
												finalList.add(fileName)
											}
										}
										dataProviderFileList = finalList.join("\n")
									}
								}
							}
						}
					stage('User Input') {
						timeout(3) {
							script {
								PARAMS = input id: 'param', message: 'List of Parameters',
								parameters: [
								//choice(choices: 'SpecialProjects', description: 'Select a package', name: 'packageName'),
								choice(choices: 'chrome', description: 'Select a Browser', name: 'browser'),
								choice(choices: env.PACKAGES, description: 'Select a Test Case', name: 'testName'),
								choice(choices: 'SQS\nSQM\nPROD\nSQP\nSQF', description: 'Select an environment', name: 'environment'),
								choice(choices: 'Trunk\nBranch\nProd', description: 'Select a Repo', name: 'repoName'),
								choice(choices: "${bsdTestUrl}", description: 'Select a Test URL', name: 'testUrl'),
								choice(choices: "${testdataFilesList}", description: 'Select a Test Data file', name: 'testDataFile'),
								choice(choices: "testng-specialprojects.xml", description: 'Select a TestNG file', name: 'testngFile')
								]
								testName = "${PARAMS['testName']}"
							}
						}
					}
					stage ('Evaluate') {
						script {
							echo "In evaluate stage"
							if (PARAMS['repoName'] == 'Trunk') {
								echo "Checking out trunk code"
								repo = 'https://ecomsvn.officedepot.com/repositories/ECOM/trunk/Automation/ClearTestMax/BSD'
							}
							else if (PARAMS['repoName'] == 'Branch'){
								echo "Checking out branch code"
								repo = 'https://ecomsvn.officedepot.com/repositories/ECOM/branches/Automation/SQM/ClearTestMax/BSD/'
							}
							else if (PARAMS['repoName'] == 'Prod'){
								echo "Checking out branch code"
								repo = 'https://ecomsvn.officedepot.com/repositories/ECOM/branches/Automation/Prod/ClearTestMax/BSD/'
							}
							if (PARAMS['testngFile'] == 'ctm-failed.xml') {
								testngFailedXml = "ctm-failed-${application}-${PARAMS['packageName']}.xml"
								testngFilePath = "/root/CTM_Automation/CTM_Reports/${testngFailedXml}"
							}
							else {
								testngFilePath = "resources/testng/${PARAMS['testngFile']}"
							}
						}
					}
					stage ("Build Parameter Info - ${application} - ${testName} - ${PARAMS['environment']} - ${PARAMS['browser']} - ${PARAMS['repoName']} - ${PARAMS['dataProviderFile']} - ${PARAMS['testUrl']}") {
						echo "${application} - ${testName} - ${PARAMS['environment']} - ${PARAMS['browser']} - ${PARAMS['repoName']} - ${PARAMS['testUrl']} - ${PARAMS['testDataFile']} - ${PARAMS['dataProviderFile']} - ${testngFilePath}"
					}
					stage ("${application} - ${testName} - ${PARAMS['environment']}  - Checkout") {
						container("${mavenContainerVersion}"){
							checkout([$class: 'SubversionSCM',
							additionalCredentials: [],
							excludedCommitMessages: '',
							excludedRegions: '',
							excludedRevprop: '',
							excludedUsers: '',
							filterChangelog: false,
							ignoreDirPropChanges: false,
							includedRegions: '',
							locations: [[cancelProcessOnExternalsFail: false,
							credentialsId: "${svnId}",
							depthOption: 'infinity',
							ignoreExternalsOption: false,
							local: '.',
							remote: "${repo}"]],
							quietOperation: true,
							workspaceUpdater: [$class: 'UpdateUpdater']])
						}
					}
					stage ("${application} - ${testName} - ${PARAMS['environment']} - Build") {
						container("${mavenContainerVersion}"){
							script {
								try {
									configFileProvider([configFile(fileId: 'mvn_settings', targetLocation: 'settings.xml')]) {
										sh """mvn --settings settings.xml -B -q test -DpackageName="${packageName}" -DtestName=${testName} -DbrowserName=${PARAMS['browser']} -DenableGrid=true -Denvironment=${PARAMS['environment']} -DtestUrl=${PARAMS['testUrl']} -DtestDataFile=${PARAMS['testDataFile']} -DtestngFile=${testngFilePath} -DgridURL=${gridURL} -Dapplication=${application} -DdataProviderFile=${PARAMS['dataProviderFile']}"""
										//cleanWs()
									}
								}
								catch (err) {
									if(manager.logContains(".*CTM Test Suite Execution Complete.*")) {
										if(manager.logContains(".*BUILD FAILURE.*")) {
											isBuildFailure = true
											currentBuild.result = 'SUCCESS'
										}
									}
									else {
										isBuildFailure = true
										currentBuild.result = 'FAILURE'
										sh "exit 1"
									}
								}
							}
						}
					}
					stage('Email User') {
						container("${mavenContainerVersion}"){
							script {
								buildUrl = "${BUILD_URL}"
								if(testName == 'CatalogTest_BSD') {
									if (!isBuildFailure) {
										sharedLib.emailUsers(isBuildFailure, "${application}", "${testName}", "${PARAMS['environment']}", "${PARAMS['browser']}", "${PARAMS['testUrl']}", buildUrl, "${constants.emailDistributionList.catalogSuccess}", "BSD Catalog Passed", "Passed",  "**/catalog-bsd.xls")
									}
									else {
										sharedLib.emailUsers(isBuildFailure, "${application}", "${testName}", "${PARAMS['environment']}", "${PARAMS['browser']}", "${PARAMS['testUrl']}", buildUrl, "${constants.emailDistributionList.catalogFailure}", "BSD Catalog Failed", "Failed", "**/catalog-bsd.xls")
									}
								}
							}
							cleanWs()
						}
					}
					stage ('Upload_Results_to_Database - Checkout') {
						container("${mavenContainerVersion}"){
							checkout([$class: 'GitSCM', branches: [[name: '*/nonprod']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: "${gitId}", url: 'https://github.com/officedepot/ctm-results-loader.git']]])
						}
					}
					stage ('Upload_Results_to_Database - Build') {
						container("${mavenContainerVersion}"){
							configFileProvider([configFile(fileId: 'mvn_settings', targetLocation: 'settings.xml')]) {
								sh """mvn --settings settings.xml --batch-mode -e install exec:java -Dexec.mainClass="com.cleartestmax.resultsloader.ResultsToDB" -Dexec.cleanupDaemonThreads=false"""
							}
						}
					}
				//}
			//}
		}
	}
