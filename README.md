# Dockerfile 
## IBM Java SDK UBI is not available on public docker yet. Use regular
## base as builder until this is ready. For reference:
## https://github.com/ibmruntimes/ci.docker/tree/master/ibmjava/8/sdk/ubi-min


### 1. Buidk base Image
	FROM ibmjava:8-sdk AS builder

### 2. Add meta information
	LABEL maintainer="IBM Java Engineering at IBM Cloud"
	
### 3. Creating app dir
	WORKDIR /app

### 4. Maven Installation
	RUN apt-get update \
  && apt-get install -y --no-install-recommends maven=3.6.0-1~18.04.1
