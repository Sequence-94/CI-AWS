
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

		put source code from github repo to codecommit repo and push

	Code Artifcat
		create an IAM user with code artifact access
		install AWS cli , configure
		export auth token
		update settings.xml file in source code top level directory 
		update pom.xml file with repo details

	Sonar Cloud
		create sonar cloud account
		generate token
		create ssm parameters with sonar details
		create build project
		update codebuild tole to access SSMparameterstore
	Create notifications for sns

	Build Process
		update pom.xml with artifact version with timestamp
		create variables in SSM => parameter store
		create build project
		update codebuild role to access SSMparameterstore
		
	Create Pipeline
		codecommit
		testcode
		build
		deploy to s3 bucket

	Test Pipeline
		
