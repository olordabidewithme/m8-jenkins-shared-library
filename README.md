Demo Project
	- Create a Jenkins Shared Library

Technologies Used
	- Jenkins, Groovy, Docker, Git, Java, Maven

Project Description
	- Create a Jenkins Shared Library to extract common build logic:
	- Create separate Git repository for Jenkins Shared Library project
	- Create functions in the JSL to use in the Jenkins pipeline
	- Integrate and use the JSL in Jenkins Pipeline (globally and for a specific project in Jenkinsfile)

1. Create the shared library
	- create groovy project
		- Name : jenkins-shared-library
			- var folder
				- each build step is its own Groovy file
				- *.groovy
			- src folder
				- helper code
			- resource folde
				- external libraries
				- non groovy file

2. Create a new package
	- src folder
		- New > Package > com.example
			- New > class
				-Docker.groovy
```bash

#!/user/bin/env groovy
package com.example

class Docker implements Serializable {

    def script

    Docker(script) {
        this.script = script
    }

    def buildDockerImage(String imageName) {
        script.echo "building the docker image..."
        script.sh "docker build -t $imageName ."
        }

    def dockerLogin() {
        script.withCredentials([script.usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
            script.sh "echo '${script.PASS}' | docker login -u '${script.USER}' --password-stdin"
        }
    }

    def dockerPush(String imageName) {
        script.sh "docker push $imageName"
    }
}

```
3. Create buildImage.groovy, buildJar.groovy, dockerLogin.groovy, dockerPush.groovy and reference the new package
	- var/buildImage.groovy
```bash
#!/user/bin/env groovy

import com.example.Docker
def call(String imageName) {
    return new Docker(this).buildDockerImage(imageName)
}
```
	- var/buildJar.groovy
```bash
#!/user/bin/env groovy

def call() {
    echo "building the application for branch $GIT_BRANCH"
    sh 'mvn package'
}
```
	- var/dockerLogin.groovy
```bash
#!/user/bin/env groovy

import com.example.Docker
def call() {
    return new Docker(this).dockerLogin()
}
```
	- var/dockerPush.groovy
```bash
#!/user/bin/env groovy

import com.example.Docker
def call(String imageName) {
    return new Docker(this).dockerPush(imageName)
}
```


4. create and push repository to GitHub
	- create a new repository in github : m8-jenkins-shared-library
	- go to local project folder
		- git init
		- git remote add origin https://github.com/olordabidewithme/m8-jenkins-shared-library.git
		- git add .
		- git commit -m "initial commit"
		- git push -u origin main



5. Integrate and use the JSL in Jenkins Pipeline Globally
	- Manage Jenkins > System > Global Pipeline Libraries > Add
		- Name : jenkins-shared-library
		- Default version : main
		- Retrieval method : Modern SCM
		- Project Repository : https://github.com/olordabidewithme/m8-jenkins-shared-library.git
		- Credentials : git-credentials

	- Reference the shared library in jenkinsfile (Globally)
		- switch to project "m8-create-jenkins-sharedlib-pipeline"
		- go to jenkinsfile

```bash
#!/user/bin/env groovy
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
```
7. Integrate and use the JSL in Jenkins Pipeline for a specific project in Jenkinsfile
```bash
#!/user/bin/env groovy

Library identifier : "jenkins-shared-library@main", retriever: modernSCM(
		[$class: 'GitSCMSource',
		remote: 'https://github.com/olordabidewithme/m8-jenkins-shared-library.git',
		credentials: 'git-credentials']) 
```



##Background

What is jenkins shared library?
	- extension to pipeline
	- has own repository
	- in groovy
	- reference shared logic in jenkinsfile
 
Why do we need it?
	- Use case 1 : micro services
		- each microservice -> one pipeline
		- each jenkinsfile of each pipeline share same logic
	- Use case 2 : multiple projects in a company
		- not use same tech stack, but ..
			- company-wide nexus repository
			- company-wide slack channel

Benefit of jenkins shared library
	- code reusability
	- easy maintenance
	- consistency and standardization
	- faster pipeline creation
	- imporved collaboration


