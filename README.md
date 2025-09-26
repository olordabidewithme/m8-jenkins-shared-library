



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

Create shared library
	- create groovy project
		- jenkins-shared-library
			- var folder
				- *.groovy
				- each step can be a groovy file
				- example
					- buildJar.groovy
						- #!/usr/bin/env groovy
						- def call(){}
					- buildImage.groovy
		
			- src folder
				- helper code
			- resource folde
				- external libraries
				- non groovy file

		- create repository in GitHub
			- jenkins-shared-library
		- on the "jenkins-shared-library" on host
			- git init
			- git remote add origin ....
			- git add .
			- git commit -m "initial commit"
			- git push

Make shared library in jenkins
	- Manage Jenkins > System > Global Pipeline Libraries > Add
		- Name : jenkins-shared-library
		- Default version : main
		- Retrieval method : Modern SCM
		- Project Repository : https://github.com/olordabidewithme/m8-jenkins-shared-library.git
		- Credentials : git-credentials

Use shared library in jenkinsfile
	- create new branch "jenkins-shared-lib" and switch to it
	- @Library('jenkins-shared-library')	
	- script { buildJar()}
	- script { buildImage()}	 
	- add DockerFile

Use parameters in shared library
	- buildImage.groovy
		- def call (String imageName)
	- buildJar.groovy
		- def call 

Extract logic into Groovy class
	- src folder
		- New > Package > com.example
			- New > class
				-Docker.groovy
```bash
#!/user/bin/env groovy
package com.example

class Docker implements Serizable {
	def script	
	Docker(script) {
		this.script = script
        }	

	def buildDockerImage(String imabe) {
		script. .....
		script.echo
		script.withCredentials
		script.sh
		${script.PASS}
	}
	def dockerLogin() {
	}

	def dockerPush(String imageName) {
	}

}
```

Reference in groovy script
	- import com.example.Docker
	- new Docker(this).buildDockerImage(imageName)

Add other groovy script
	- dockerLogin.groovy
	- dockerPush.groovy


Project scope shared library
	- Library identifier : "jenkins-shared-library@main", retriever: modernSCM( )
	- @Library('jenkins-shared-library@2.0')
