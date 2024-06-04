// Uses Declarative syntax to run commands inside a container.
@Library('cloudbees-shared-library@dev') _
def podYaml = libraryResource "podTemplates/python3.10.yaml"
pipeline {
    agent {
        kubernetes {
            yaml podYaml
        }
    }
    stages {
        stage('install requirements.txt') {
            steps {
                container("base-agent"){
                    script{
                        python.installRequirements()
                    }
                }
                
            }
        }
        stage('run pytest') {
            steps {
                container("base-agent"){
                    script{
                        python.test()
                    }    
                }
                
            }
        }
        stage('sonarqube scan') {
            steps {
                container("sonarqube"){
                    script{
                        sonarqube.scan()
                    }
                }
                
            }
        }
        stage('snyk scan') {
            steps {
                container("base-agent"){
                    script{
                        snyk.scan()
                    }
                }
                
            }
        }
        stage('artifact upload') {
            steps {
                container("kaniko"){
                    script{
                        kaniko.upload()
                    }
                }
                
            }
        }
    }
}
