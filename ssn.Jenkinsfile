// Uses Declarative syntax to run commands inside a container.
//@Library('cloudbees-shared-library@master')_
library identifier: 'cloudbees-shared-library@dev',
	retriever: modernSCM([
		$class: 'GitSCMSource', 
		credentialsId: 'svcgtsdcatpipeline', 
		remote: 'https://git.us.aegon.com/AGT-DCAT/cloudbees-shared-library.git'
	])
def podYaml = libraryResource "podTemplates/python3.10.yaml"
// Here's an example of a pipeline script using Jenkins declarative pipeline syntax for the scenario described
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                // For Java/Maven Docker Images
                sh 'git clone https://git.us.aegon.com/Cloud-IAM-Team/gts-servicecatalog-engine.git'
                sh 'mvn clean install'
                sh 'docker build -t java_app .'

                // For Python Images
                sh 'git clone <repository_url>'
                sh 'pip install -r requirements.txt'
                sh 'docker build -t python_app .'
            }
        }

        stage('Test') {
            steps {
                // For Java/Maven Docker Images
                sh 'mvn test'
                // Additional steps like integration tests, static code analysis, security scans
                
                // For Python Images
                sh 'pytest'
                // Additional steps like linting, security scans
            }
        }

        stage('sonarQube') {
            steps {
                // for both Java/Maven Docker Images and Python Images
                // Run SonarQube analysis
                sh 'mvn sonar:sonar'
            }
        }
        
        stage('Security Scans') {
            steps {
                // Snyk security scans
                // For Java/Maven Docker Images
                sh 'snyk test --all-projects'

                // For Python Images
                sh 'snyk test'
            }
        }

        stage('Deploy') {
            steps {
                // For both Java/Maven Docker Images and Python Images
                sh 'docker tag java_app <registry_url>/java_app:$BUILD_NUMBER'
                sh 'docker tag python_app <registry_url>/python_app:$BUILD_NUMBER'
                
                // Additional steps like pushing images to registry, deploying to staging/production
            }
        }

        stage('Publish to Nexus') {
            steps {
                // FOr both Java/Maven Docker Images and Python Images
                sh 'mvn deploy'
            }
        }
    }
}