library identifier: 'cloudbees-shared-library@feat/base-pipeline',
	retriever: modernSCM([
		$class: 'GitSCMSource', 
		credentialsId: 'svcgtsdcatpipeline-token', 
		remote: 'https://git.us.aegon.com/AGT-DCAT/cloudbees-shared-library.git'
	])
def podYaml = libraryResource "podTemplates/python3.10.yaml"
pipeline {
    agent {
        kubernetes {
            yaml podYaml 
        }
    }
 
    stages {
        stage('Checkout Repo') {
            steps {
                checkout scm
                echo 'checkout scm'
            }
        }
        stage('Tag Release') {
                steps {
                    script {
                    def releaseVersion = '2.0.0'// Your desired version number
                            git remote -v
                    }
                }
            }
        stage('Build') {
            steps {
                container("base-agent"){
                    script{
                        sh 'pip install -r src/requirements-dev.txt'
                    }
                }  
            }
        }

        stage('Unit Tests') {
            steps {
                container("base-agent"){
                    script{
                        sh 'python src/tests/test_common.py'
                        sh 'python src/tests/test_incident.py'
                    }
                }  
            }
        }

        stage('sonarqube scan') {
            steps {
                container("sonarqube"){
                    script{
                        withCredentials([string(credentialsId: 'sonar_token', variable: 'sonar_token')]) {
                            sonar.scan()
                        }
                    }
                }
            }
        }

        stage('Snyk Test using Snyk CLI') {
            steps {
                container("snyk"){
                    script{
                        withCredentials([string(credentialsId: 'snyk-cloudbees', variable: 'SNYK_TOKEN')]) {
                            snyk.scan()
                            // sh "snyk code test --org=1f07b5ff-3d7f-43a3-861a-7f6f5bf10e49"
                        }
                    }
                }
            }
        }

        // stage('Snyk Test using Snyk Security Plugin') {
        //     steps {
        //         snykSecurity(
        //             snykInstallation: 'snyk@latest',
        //             snykTokenId: 'snyk-api-token',
        //             // place other optional parameters here, for example:
        //             additionalArguments: '--all-projects'
        //         )
        //     }
        // }

        // stage('Snyk Test using Snyk Security Plugin') {
        //     steps {
        //         container("snyk"){
        //             script{
        //                 snykSecurity(
        //                     snykInstallation: 'snyk@latest',
        //                     snykTokenId: 'snyk-api-token',
        //                     // place other optional parameters here, for example:
        //                     additionalArguments: '--all-projects'
        //                 )
        //             }
        //         }
        //     }
        // }

        stage('artifact upload') {
            steps {
                container("base-agent"){
                    script{
                        sh 'ls'
                        withCredentials([usernamePassword(credentialsId: 'nexus-credential', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                            nexus.create_file()
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
