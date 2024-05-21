
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

Scenario :
	Agile SDLC
	Developers make regular code changes.
	These commits need to be Built & Tested regularly

	Usually, Build & Release Team will do this job
	Or in a small industry, it will be a Developers' responsibility to merge and integrate code.

Problem :
	In an Agile SDLC, there will be frequent code commits and pull requests.
	Developers will be dependent on the Build & release team, usually to test the code and move it
	to the next level in the release cycle, but this is not done so frequently.
	Which means bugs and errors may accumulate.

	Developers will need to rework to fix these bugs and errors which might be time-consuming.
	The developers will need to fix the problems as fast as its building in order to keep up, however,
	currently, we have a Manual Build & Release process and Inter Team Dependencies(for instance approvals and authentication systems in place).

Fix:
	Build & Test For Every Commit.
	This means that as soon as there is a code change, the code needs to be built and tested simultaneously.
	This is not possible with the MANUAL build and release process so we need to AUTOMATE this process.
	Developers should be notified instantaneously for every build status in case there is a build failure, such as 
	not passing the quality gates or there are bugs so they can fix it then and there.
	This is called Continuous Integration Process.
	I will use AWS Cloud Services so that I don't have to manage CI Servers myself such as Jenkins. This removes some overhead.

Benefits of Continous Integration Pipeline on AWS:
	Short Mean Time To Repair - which leads to reduced downtime.
	Agile - which leads to iterative development, collaboration, and flexible response to changes
	No human intervention - which leads to speed, reduced human error, more efficiency
	No ops - which means we eliminate the need for dedicated operations teams through automation and cloud services.
	Fault isolation -  which means we can isolate failures to specific services without affecting the entire system, 
				 makes it easier to diagnose and fix problems.


AWS SERVICES:
	Code Commit - our version control system repository, basically like GitHub
	Code Artifact - maven will download dependencies from the code artifact repository
	Code Build  - our build service from AWS(maven build, code analysis,sonarqube analysis)
	Code Deploy - our artifact deployment service(to S3 bucket)
	SONARCLOUD - our sonarqube cloud-based tool. We push all of our results from code build to Sonarcloud
	CHECKSTYLE - our code analysis tool
	CODEPIPELINE - our service that integrates all jobs.


FLOW OF EXECUTION:

	Login to the AWS account

	Code Commit
		create codecommit repo
		create iam user with codecommit policy
		generate ssh keys locally
		exchange keys with IAM user
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
		
