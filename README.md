Pre-requisite
=============
Ensure two secrets are exported in your local machine

For Mac
-------
In a terminal, type

export FIRST_SECRET='This is my first secret'

export SECOND_SECRET='This is my second secret'

For Windows
-----------
Go to Edit the System Environment Variables,

Click on Advanced--> Environment Variables--> click on New

Provide Varialble name as FIRST_SECRET and Variable Value as This is my first secret

Similarly Provide variable name as SECOND_SECRET and Variable Value as This is my second secret

Using Maven
===========
To build and run locally,

change the directory to the project root folder then run :

mvn clean install

change the directory to the target folder

java -jar cirrusjavacrontemplate-1.0-SNAPSHOT.jar


Using Docker
============
Ensure docker is installed 

Change the directory to the project root folder then run :

docker build -t cirrusjavacrontemplate .

docker run --env FIRST_SECRET --env SECOND_SECRET cirrusjavacrontemplate

After Local / Docker Build
==========================
Verify the end points available at

Health : http://localhost:8080/rest/health

Application : http://localhost:8080/


# Dockerfile Instructions

### IBM Java SDK UBI is not available on public docker yet. Use regular base as builder until this is ready. For reference: https://github.com/ibmruntimes/ci.docker/tree/master/ibmjava/8/sdk/ubi-min

### 1. Build base Image

	FROM ibmjava:8-sdk AS builder

### 2. Add meta information

	LABEL maintainer="IBM Java Engineering at IBM Cloud"
	
### 3. Creating app dir
	WORKDIR /app

### 4.Maven Installation
	RUN apt-get update \
  	&& apt-get install -y --no-install-recommends maven=3.6.0-1~18.04.1

### 5. Copy pom.xml
	COPY pom.xml .
	RUN mvn -N io.takari:maven:wrapper -Dmaven=3.5.0

### 6. Set variable
	ARG bx_dev_user=root
	ARG bx_dev_userid=1000
	RUN export BX_DEV_USER=$bx_dev_user
	RUN export BX_DEV_USERID=$bx_dev_userid
	RUN if [ $bx_dev_user != "root" ]; then useradd -ms /bin/bash -u $bx_dev_userid $bx_dev_user; fi
### Multi-stage build. New build stage that uses the UBI as the base image. In the short term, we are using the OpenJDK for UBI. Long term, we will use the IBM Java Small Footprint JVM (SFJ) for UBI, but that is not in public Docker at the moment.(https://github.com/ibmruntimes/ci.docker/tree/master/ibmjava/8/sfj/ubi-min)

### 7. Java base Image
	FROM adoptopenjdk/openjdk8:ubi-jre
	
### 8. Copy and rename jar file
	RUN mkdir /opt/app
	COPY --from=builder /app/target/cirrusjavacrontemplate-1.0-SNAPSHOT.jar /opt/app/app.jar

### 9. Set Env port
	ENV PORT 8080

### 10. Export port
	EXPOSE 8080

### 11. To run jar file
	ENTRYPOINT [ "sh", "-c", "java -jar /opt/app/app.jar" ]
