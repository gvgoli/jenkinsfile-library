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

        // stage('Build') {
        //     steps {
        //         container("maven"){
        //             script{
        //                 sh 'mvn clean install'
        //                 sh 'mvn package'
		// 	            sh 'ls'
        //             }
        //         }  
        //     }
        // }

        stage('sonarqube scan') {
            steps {
                container("sonarqube"){
                    script{
                        withCredentials([string(credentialsId: 'sonar_token', variable: 'sonar_token')]) {
                            sonar1.scan()
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
                        }
                    }
                }
            }
        }

        stage('artifact upload') {
            steps {
                container("base-agent"){
                    script{
                        sh 'ls'
                        withCredentials([usernamePassword(credentialsId: 'nexus-credential', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                            nexus1.create_file()
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