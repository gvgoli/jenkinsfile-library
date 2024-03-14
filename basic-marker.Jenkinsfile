// Uses Declarative syntax to run commands inside a container.
@Library('cloudbees-shared-library@dev') _
def podYaml = libraryResource "podTemplates/python3.10.yaml"
def properties
pipeline {
    agent {
        kubernetes {
            yaml podYaml
        }
    }
    stages {
        stage('read markerfile') {
            steps {
                container("base-agent"){
                    script{
                        properties = readYaml file: "./ci.yaml"
                    }
                }
                
            }
        }
        stage('install requirements.txt') {
            steps {
                container("base-agent"){
                    script{
                        python.installRequirements("${properties.req_file}")
                    }
                }
                
            }
        }
        stage('run pytest') {
            steps {
                container("base-agent"){
                    script{
                        python.test("${properties.pytest_args}")
                    }    
                }
                
            }
        }
        stage('sonarqube scan') {
            steps {
                container("sonarqube"){
                    script{
                        sonarqube.scan("${properties.sonarqube.project_key}")
                    }
                }
                
            }
        }
        stage('snyk scan') {
            steps {
                container("base-agent"){
                    script{
                        snyk.scan("${properties.synk.project_key}")
                    }
                }
                
            }
        }
        stage('black duck scan') {
            steps {
                container("base-agent"){
                    script{
                        black_duck.scan("${properties.black_duck.project_key}")
                    }
                }
                
            }
        }
        stage('artifact upload') {
            steps {
                container("kaniko"){
                    script{
                        kaniko.upload("${properties}")
                    }
                }
                
            }
        }
    }
}
