library identifier: 'cloudbees-shared-library@dev',
	retriever: modernSCM([
		$class: 'GitSCMSource',
		credentialsId: 'svcgtsdcatpipeline-token',
		remote: 'https://git.us.aegon.com/AGT-DCAT/cloudbees-shared-library.git'
	])
def podYaml = libraryResource "podTemplates/python3.10.yaml"
def ciConfig = [:]  // Define ciConfig as an empty map at the beginning of your script
// Cloudbees CI UI of pipeline - Configuration -> Properties -> create a Environment Variables -> DEBUG_MODE=true
def debugPrint(String message) {
    if (env.DEBUG_MODE == 'true') {
        echo message
    }
}

pipeline {
    agent {
        kubernetes {
            yaml podYaml
        }
    }

    stages {
        stage('Init') {
            steps {
                container("base-agent"){
                    setEnv()
                    script{
                        logger.info("Pulling Source Code")
                        checkout scm
                        logger.info("Successfully Cloned")
                    }
                }
            }
            post {
                failure {
                    container('base-agent') {
                        script {
                            logger.info("Error Pulling Source Code")
                        }
                    }
                }
            }
        }

        stage('Read YAML') {
            steps {
                script {
                    if (fileExists('ci.yaml')) {
                        try {
                            // Use readYaml to read the ci.yaml file from the workspace directory
                            ciConfig = readYaml(file: "ci.yaml") // Read the ci.yaml file into the ciConfig map
                            setEnv.setStages(ciConfig)
                            sh 'cat ci.yaml'
                            if (env.DEBUG_MODE == 'true') {
                                debugPrint('Debug mode is on.')
                                debugPrint("Current directory: ${pwd()}")
                                sh 'ls -l ci.yaml'
                                debugPrint("ciConfig: ${ciConfig}")  // Print the contents of the ciConfig map
                            }
                        } catch (Exception e) {
                            error "Error reading ci.yaml: ${e.message}"
                        }
                    } else {
                        error 'ci.yaml does not exist'
                    }
                }
            }
        }

        stage('Build') {
            // when {
            //     expression { env.Build == 'true' }
            // }
            steps {
                script {
                    debugPrint("env.BUILD: ${env.BUILD}")
                    debugPrint("STAGE_NAME: ${env.STAGE_NAME}")
                    if (env.BUILD == env.STAGE_NAME) {
                        // Execute the custom build steps from ci.yaml
                        echo 'Running custom build steps...'
                        container('base-agent') {
                            ciConfig.Build.each { step ->
                                sh step
                            }
                        }
                    } else {
                        // Default to a standard Maven build
                        echo 'No Build stage provided, Skipping.'
                        // container('maven') {
                        //     sh 'mvn clean install'
                        // }
                    }
                }
            }
        }

        stage('Test') {
            // when {
            //     expression { env.Test == 'true' }
            // }
            steps {
                script {
                    debugPrint("env.TEST: ${env.TEST}")
                    debugPrint("STAGE_NAME: ${env.STAGE_NAME}")
                    if (env.TEST == env.STAGE_NAME) {
                        // Execute the custom build steps from ci.yaml
                        echo 'Running custom test steps...'
                        container('base-agent') {
                            ciConfig.Test.each { step ->
                                sh step
                            }
                        }
                    } else {
                        // Default to a standard Maven build
                        echo 'No Test stage provided, Skipping.'
                        // container('maven') {
                        // sh 'mvn test'
                        // }
                    }
                }
            }
            // post {
            //     always {
            //     junit '**/target/surefire-reports/TEST-*.xml'
            //     }
            // }
        }

        stage('Security Scans') {
            parallel {
                stage('SonarQube Scan') {
                    // options {
                    //     lock(resource: "${properties.ci.appName}-${env.BRANCH_NAME}-" + "Sonarqube", inversePrecedence: true)
                    // }
                    steps {
                        container("sonarqube"){
                            script{
                                logger.info("Sonar Scan Initiated")
                                withCredentials([string(credentialsId: 'sonar_token', variable: 'sonar_token')]) {
                                    sonar.scan()
                                }
                            }
                        }
                    }
                    post {
                        success {
                            container('base-agent') {
                                script {
                                    // jira.postComment("sonarqube", "passed")
                                    logger.info("Sonar Scan Passed")
                                }
                            }
                        }
                        failure {
                            container('base-agent') {
                                script {
                                    // jira.postComment("sonarqube", "failed")
                                    logger.info("Sonar Scan Failed")
                                }
                            }
                        }
                    }
                }

                stage('Snyk Security Report') {
                    // options {
                    //     lock(resource: "${properties.ci.appName}-${env.BRANCH_NAME}-" + "Snyk", inversePrecedence: true)
                    // }
                    steps {
                        container('base-agent') {
                            script{
                                logger.info("Snyk Security Report Initiated")
                                    snyk.securityReport()
                            }
                        }
                    }
                    post {
                        success {
                            container('base-agent') {
                                script {
                                    // jira.postComment("snyk", "passed")
                                    logger.info("Snyk Security Report Passed")
                                }
                            }
                        }
                        failure {
                            container('base-agent') {
                                script {
                                    // jira.postComment("snyk", "failed")
                                    logger.info("Snyk Security Report Failed")
                                }
                            }
                        }
                    }
                }

                stage('Snyk Code') {
                    // options {
                    //     lock(resource: "${properties.ci.appName}-${env.BRANCH_NAME}-" + "Snyk", inversePrecedence: true)
                    // }
                    steps {
                        container('snyk') {
                            script{
                                logger.info("Snyk Code Scan Initiated")
                                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                                    withCredentials([string(credentialsId: 'snyk_global_token', variable: 'snyk_global_token')]) {
                                        snyk.auth()
                                        snyk.code()
                                    }
                                }
                                recordIssues  tool: sarif(name: 'Snyk Code', id: 'snyk-code', pattern: 'results-code.sarif')
                            }
                        }
                    }
                    post {
                        success {
                            container('base-agent') {
                                script {
                                    // jira.postComment("snyk", "passed")
                                    logger.info("Snyk Code Scan Passed")
                                }
                            }
                        }
                        failure {
                            container('base-agent') {
                                script {
                                    // jira.postComment("snyk", "failed")
                                    logger.info("Snyk Code Scan Failed")
                                }
                            }
                        }
                    }
                }
            }
        }

        stage('Artifact Upload') {
            when {
                expression { env.ARTIFACT == 'true' }
            }
            steps {
                container("base-agent"){
                    script{
                        sh 'ls'
                        withCredentials([usernamePassword(credentialsId: 'nexus-credential', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                            nexus.createRepositoryAndUpload()
                        }
                    }
                }
            }
            post {
                failure {
                    container('base-agent') {
                        script {
                            logger.info("Nexus Publish Failed")
                        }
                    }
                }
            }
        }
    }
    post{
        success {
            echo "Pipeline is complete"
        }
        failure {
            echo "Pipeline is Failed"
        }
    }
}
