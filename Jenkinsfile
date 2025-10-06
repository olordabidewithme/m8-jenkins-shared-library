import library : @Library('jenkins-shared-library')

pipeline {

	agent any

    	tools {
        	maven 'maven-3.9.2'
    	}	
	
	stage("build jar") {
		steps {
			script{
				buildJar()
			}
		}
	}
	stage("build and push image") {
		steps {
			script{
				buildImage 'olordabidewithme/demo-app-3.0'
				dockerLogin()
				dockerPush 'olordabidewithme/demo-app-3.0'
			}
		}
	}
        stage("deploy") {
            steps {
                script {
                    gv.deployApp()
                }
            }
        }  
}
