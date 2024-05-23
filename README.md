![Screen Shot 2024-05-21 at 16 38](https://github.com/Sequence-94/CI-AWS/assets/53806574/724a6098-f77e-421f-8de2-8fb026d26229)
![Continous Integration On AWS Cloud](https://github.com/Sequence-94/CI-AWS/assets/53806574/9f4171fc-9b17-411a-bcfe-ad3b5d038c39)


# Prerequisites
#
- JDK 11 
- Maven 3 
- MySQL 8

# Technologies 
- Spring MVC
- Spring Security
- Spring Data JPA
- Maven
- JSP
- Tomcat
- MySQL
- Memcached
- Rabbitmq
- ElasticSearch
# Database
Here,we used Mysql DB 
sql dump file:
- /src/main/resources/db_backup.sql
- db_backup.sql file is a mysql dump file.we have to import this dump to mysql db server
- > mysql -u <user_name> -p accounts < db_backup.sql

# Scenario :
	Agile SDLC
	Developers make regular code changes.
	These commits need to be Built & Tested regularly

	Usually, Build & Release Team will do this job
	Or in a small industry, it will be a Developers' responsibility to merge and integrate code.

# Problem :
	In an Agile SDLC, there will be frequent code commits and pull requests.
	Developers will be dependent on the Build & release team, usually to test the code and move it
	to the next level in the release cycle, but this is not done so frequently.
	Which means bugs and errors may accumulate.

	Developers will need to rework to fix these bugs and errors which might be time-consuming.
	The developers will need to fix the problems as fast as its building in order to keep up, however,
	currently, we have a Manual Build & Release process and Inter Team Dependencies(for instance approvals and authentication systems in place).

# Fix:
	Build & Test For Every Commit.
	This means that as soon as there is a code change, the code needs to be built and tested simultaneously.
	This is not possible with the MANUAL build and release process so we need to AUTOMATE this process.
	Developers should be notified instantaneously for every build status in case there is a build failure, such as 
	not passing the quality gates or there are bugs so they can fix it then and there.
	This is called Continuous Integration Process.
	I will use AWS Cloud Services so that I don't have to manage CI Servers myself such as Jenkins. This removes some overhead.

# Benefits of Continous Integration Pipeline on AWS:
	Short Mean Time To Repair - which leads to reduced downtime.
	Agile - which leads to iterative development, collaboration, and flexible response to changes
	No human intervention - which leads to speed, reduced human error, more efficiency
	No ops - which means we eliminate the need for dedicated operations teams through automation and cloud services.
	Fault isolation -  which means we can isolate failures to specific services without affecting the entire system, 
				 makes it easier to diagnose and fix problems.


# AWS SERVICES:
	Code Commit - our version control system repository, basically like GitHub
	Code Artifact - maven will download dependencies from the code artifact repository
	Code Build  - our build service from AWS(maven build, code analysis,sonarqube analysis)
	Code Deploy - our artifact deployment service(to S3 bucket)
	SONARCLOUD - our sonarqube cloud-based tool. We push all of our results from code build to Sonarcloud
	CHECKSTYLE - our code analysis tool
	CODEPIPELINE - our service that integrates all jobs.


# FLOW OF EXECUTION:

	Login to the AWS account

	Code Commit
		create codecommit repo - Creating the code repository is fairly simple when using CodeCommit AWS Console. 
  		I simply entered the name for the repo "vprofile-code-repo" and left evrything as default.
![Screen Shot 2024-05-21 at 14 37](https://github.com/Sequence-94/CI-AWS/assets/53806574/133f0762-c85a-4a53-92a6-b39cb56ff2f0)

		create IAM-user with codecommit policy this will allow me access into this repository locally with the correct
  		priviledges to my codecommit repository from my computer. I chose the user-name "vprofile-code-admin" for IAM-user
    		and chose to "attach policies directly". I specified permissions for my codecommit service and allowed all actions under the sun.
      		I also added an ARN which was my vprofile-code-repo. So this IAM user only has access to the repo and is allowed to do anything to that specific
		repo. I named this policy "vprofile-repo-fullAccess" and this is the policy I will attach to the IAM user.
![Screen Shot 2024-05-21 at 14 50](https://github.com/Sequence-94/CI-AWS/assets/53806574/912fea61-7947-496e-a316-91b3b64eadfe)
Here is my user with codecommit full access:
![Screen Shot 2024-05-21 at 14 54](https://github.com/Sequence-94/CI-AWS/assets/53806574/d7e07669-4945-44bc-8454-0f2f5281152b)

I then switched to "security credentials" tab in order to create a security access keys for cli access. 
![Screen Shot 2024-05-21 at 14 58](https://github.com/Sequence-94/CI-AWS/assets/53806574/f538844f-f9d1-41a5-b2bf-911aa462dddb)

		generate ssh keys locally 
![Screen Shot 2024-05-21 at 15 06](https://github.com/Sequence-94/CI-AWS/assets/53806574/781364e5-500c-4d86-a90d-918247bd04cf)
		upload public ssh key into SSH public keys for AWS CodeCommit
![Screen Shot 2024-05-21 at 15 17](https://github.com/Sequence-94/CI-AWS/assets/53806574/f6a3a127-2e40-4ef7-9ac7-d81f01178ffc)

		We need to creat an SSH Config file that essentially says that when we access our code commit repo then use 
  		the previously generated SSH keys for access. So when we do SSH this config file will get activated , it will see
    		that we're going to use codecommit so it will use the private ssh key we generated previously and the user id for iam
![Screen Shot 2024-05-21 at 15 32](https://github.com/Sequence-94/CI-AWS/assets/53806574/bea3a8c1-8522-4ff0-aa57-3672cf5cffb6)
		Testing whether the config file works and is able to authenticate us.
![Screen Shot 2024-05-21 at 15 37](https://github.com/Sequence-94/CI-AWS/assets/53806574/f63c60bf-88cb-4fbf-900a-128b6f2b0f63)
		Incase of any error I can run this code to troubleshoot:
  		"ssh -v git-codecommit.us-east-1.amazonaws.com"
    		For exmaple I can see which keys are being used, the config file being used.

		Push source code from github repo to codecommit repo and push:
  		First I cloned the github repo to my local desktop:
![Screen Shot 2024-05-21 at 16 32](https://github.com/Sequence-94/CI-AWS/assets/53806574/3ac098c3-3c59-4fee-82ca-d7639b5ce4fc)
   
I then removed the [remote "origin"] from github and relaced it with the one given to me in Clone URL -> CLone SSH in code commmit repo console.
![Screen Shot 2024-05-21 at 16 36](https://github.com/Sequence-94/CI-AWS/assets/53806574/02c43b66-5ce7-4de1-87dd-d3ff35dbbb8a)

After the "git push origin --all" command:
![Screen Shot 2024-05-21 at 16 38](https://github.com/Sequence-94/CI-AWS/assets/53806574/d2c1aadf-c56c-45ab-8852-a37bfd01c210)
All the files from github are now in my code commit repo


	Code Artifcat - This is a repository in AWS used to store dependencies for build tools such as Maven. So instead of downlaoding dependencies from the internet , it will download them 			from AWS code artifact repository which is secure.
 	Creating a CodeArtifact in AWS console is fairly easy - I simply  gave it a meaningful name "vprofile-maven-repo"
  	Pick my build tool from Public upstream repositories "maven-central-store"
   	Select your account
    	Entered a Domain name and left evrything else as default
![Screen Shot 2024-05-22 at 06 24](https://github.com/Sequence-94/CI-AWS/assets/53806574/979f986d-4e3d-44ec-8188-de6ea35fad98)

At this point I have two repositories, one for my code repository and another for maven dependencies repository:
![Screen Shot 2024-05-22 at 06 29](https://github.com/Sequence-94/CI-AWS/assets/53806574/662f905d-00c4-4b69-a4c4-ee2a1cc09d22)

		Now I need to make sure that my maven command connects to this specific repository.
  		
	
		create an IAM user with code artifact access
		install AWS cli , configure
		export auth token
		update settings.xml file in source code top level directory 
		update pom.xml file with repo details

	Sonar Cloud
		create sonar cloud account
![Screen Shot 2024-05-22 at 06 59](https://github.com/Sequence-94/CI-AWS/assets/53806574/b7784bbf-d912-44b4-9a37-064c28393e59)
  
		generate token
 Under MyAccount in Security Tab
 ![Screen Shot 2024-05-22 at 07 02](https://github.com/Sequence-94/CI-AWS/assets/53806574/68be70de-08b7-4a83-b3e1-e8cacf44775f)

  		Thereafter I created an organisation and gave it a unque name.
    		Under that specific organisation I then created "Anaylze Projects" again I gave it a unique identifer.
![Screen Shot 2024-05-22 at 07 16](https://github.com/Sequence-94/CI-AWS/assets/53806574/e7697448-a45c-47dd-baa1-a45515ca6ec6)
I saved this information in my sticky notes for use later.
      
		create ssm parameters with sonar details - this is to prevent exposing my authentication tokens since I may not hardcode senstive information into my build_buildspec.yml file
  		We can use Systems Manager in AWS to do this.
![Screen Shot 2024-05-22 at 07 28](https://github.com/Sequence-94/CI-AWS/assets/53806574/c7844fe3-ce67-4a7c-8ee5-5dc6d44fd41a)
 
		create build project
		update codebuild tole to access SSMparameterstore
	Create notifications for sns

	Build Project - Using CodeBuild in AWS console
		update pom.xml with artifact url - this tells maven where to fetch the dependencies
![Screen Shot 2024-05-22 at 07 45](https://github.com/Sequence-94/CI-AWS/assets/53806574/f85938ef-a4e1-4bcd-9f13-8d37ad07ab36)

This mirror configuration tells Maven to redirect all requests to the specified AWS CodeArtifact repository. Requets must go through this mirror.
![Screen Shot 2024-05-22 at 07 53](https://github.com/Sequence-94/CI-AWS/assets/53806574/7edec5e1-7b6b-44a6-bbc1-3ca83a2307da)

The three files —buildspec.yml, settings.xml, and pom.xml—are used together to manage the build, dependency management, and configuration of a Java project using AWS CodeBuild and Apache Maven.
The buildspec file defines build instructions for AWS CB and includes commands that will be exec at each phase(install,pre_build,build). It also manages environment variables via SMPS.
Below is a snippet:
![Screen Shot 2024-05-22 at 08 06](https://github.com/Sequence-94/CI-AWS/assets/53806574/c98215fb-1da7-42e2-ab53-e520b724768e)

The settings.xml configures Maven settings, including repository credentials and profiles. It ensures Maven can authenticate and access the required repositories, particularly AWS CodeArtifact.
Below is a snippet:
![Screen Shot 2024-05-22 at 08 07](https://github.com/Sequence-94/CI-AWS/assets/53806574/2eb397bc-95fd-4066-8a83-611771a2f53f)

The pom.xml  which stands for Project Object Model file defines all project dependencies,plugins and where to find them. Below is a snipet:
![Screen Shot 2024-05-22 at 08 11](https://github.com/Sequence-94/CI-AWS/assets/53806574/ec51f88b-ecf4-44fc-8511-fb0bacc94d74)

ERROR:
![Screen Shot 2024-05-22 at 09 41](https://github.com/Sequence-94/CI-AWS/assets/53806574/b351c15f-2df9-4d39-b010-71a9f68987ad)
This error took the whole day to fix mainly due to the fact that the developer of the app had said that it only runs on jdk8 or 11 only so upgrading the Java version was not even an opeion I considred even though I was very much aware that would have been the simple fix but the app might not run.
After doing some googling and research it it seems that was the quick fix. 
1st attempt: I tried to add some dependencies and even force depricate dependecies but that did not work.
2nd attempt: I switch from jdk11 to jdk8
3rd attempt: I tried use the -e -x to see logs and errors but it was futile because the error was pretty much a compatibility issue
4th attempt: I tried different version of sonar-mavin-plugin - ones that were comptaible with either jdk8 or jdk11 depending on which jdk i was attempting at the time
5th attempt: tried changing maven6.0->7.0
6th attempt: I tried upgrading to jdk17 and it seemed to have resolved the issue
![Screen Shot 2024-05-22 at 17 36](https://github.com/Sequence-94/CI-AWS/assets/53806574/d491f702-318c-46fb-ae8f-36859520cd07)

The main idea was to upload the code analysis result to SonarCloud:
![Screen Shot 2024-05-22 at 17 40](https://github.com/Sequence-94/CI-AWS/assets/53806574/f5db5e88-3cb4-4ce8-8f1b-6c1c972115ae)

  		version with timestamp
		create variables in SSM => parameter store
		create build project
		update codebuild role to access SSMparameterstore

Previously we built for analysis and to retireve dependencies from codeartifact.
Now we run a build process to build our artifact from previous build , below is a snippet of the build yml file.
![Screen Shot 2024-05-23 at 10 58](https://github.com/Sequence-94/CI-AWS/assets/53806574/b64d0695-7e58-4fd1-8ab9-f4b0930dd1b3)

This build will need permission to to codeartifact so we need to change the policy like below:
![Screen Shot 2024-05-23 at 11 07](https://github.com/Sequence-94/CI-AWS/assets/53806574/4a56effb-94a7-453d-8fae-9479c245b53f)

ERROR:
![Screen Shot 2024-05-23 at 11 11](https://github.com/Sequence-94/CI-AWS/assets/53806574/20043302-c9a2-4691-9311-404bf89fbc54)
This indicates the build process is unable to locate the build_buildspec.yml file inside the aws-files, so I resolved it by placing the build file in the root path.
![Screen Shot 2024-05-23 at 11 43](https://github.com/Sequence-94/CI-AWS/assets/53806574/5419d718-a444-40ed-8134-e32a2f5136a8)

	Create Pipeline - lets tie everything up:
 First create an S3 bucket with a folder in which we will store the artifacts:
 ![Screen Shot 2024-05-23 at 11 50](https://github.com/Sequence-94/CI-AWS/assets/53806574/d7a108f9-8e2f-4488-bce7-88977ebba07a)

	Second create an SNS topic then create a subsprition with your email as the protocol.
![Screen Shot 2024-05-23 at 11 54](https://github.com/Sequence-94/CI-AWS/assets/53806574/746160dd-498c-4e93-97d6-124d790fb430)
Confirm subscription.

	Third- Create Pipeline
 	Give it a name, the default service role will suffice.
  	Give your "CodeCommit" repository as the source provider -  because it is to observe the commit, whenever the developer makes a commit
   	the pipeline will detect and trigger the pipeline.
    	Pick CloudWatch Events so that it triggers when theres a commit as opposed to periodiclly checking for changes even though there might not be any.
     	The Build Provider can be Jenkins but we have been using CodeBuil for this project.
![Screen Shot 2024-05-23 at 12 07](https://github.com/Sequence-94/CI-AWS/assets/53806574/cd91694c-b6d6-43f7-87fd-883125077c7e)


		codecommit
		testcode
		build
		deploy to s3 bucket

	Test Pipeline
		
