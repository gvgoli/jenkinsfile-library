library identifier: 'cloudbees-shared-library@dev',
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
        // stage('Build and Package') {
        //     steps {
        //         script {
        //             sh '''tar -cvf myap-1.0.tar .'''

        //         }
        //     }
        // }           
        stage('Upload to Nexus') {
            steps {
                container("base-agent"){
                    script {
                git 'https://git.us.aegon.com/Cloud-IAM-Team/gts-servicecatalog-engine.git'  
                    withCredentials([usernamePassword(credentialsId: 'nexus-credential', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                    sh '''
		                tar -cvf artifact-${BUILD_NUMBER}.tar .
		                curl -v -u $USERNAME:$PASSWORD --upload-file ./artifact-${BUILD_NUMBER}.tar https://sonatype.agtservices-nonprod.aegon.io/repository/test-1/AEGON/CLOUD-IAM/${BUILD_NUMBER}/artifact-${BUILD_NUMBER}.tar
	                    '''                     
                        }
                    }
                }
            }
        }
    }
}