# Task INC-14

 1. [Goals](#goals)
 2. [Description](#description)
 3. [Requirements](#requirements)
 4. [Duration](#duration)
 5. [General plan](#general-plan)
 6. [Usage](#usage)
 7. [Verification](#verification)
 8. [Remove environment](#remove-environment)
 9. [Known issues](#known-issues)


## Goals

 Setup small docker swarm cluster


## Description

 Set up cluster consisting of 1 master and 1 worker (t2.micro). This cluster will be used for additional deployment tasks.


## Requirements

 1. Linux host with [Docker Machine installed](https://docs.docker.com/machine/install-machine/).
 2. AWS account with AmazonEC2FullAccess permissions.


## Duration

 This task may be accomplished in for about 10 minutes.


## General Plan

 1. Prepare environment.
 2. Create two Docker instances.
 3. Configure first instance as Docker Swarm manager.
 4. Configure second instance as Docker Swarm cluster nodes.
 5. Verify cluster status.


## Usage


### Create two Docker instances

 1. Provide AWS credentials and region:
	```bash
	export AWS_ACCESS_KEY_ID=access key
	export AWS_SECRET_ACCESS_KEY=secret access key
	export AWS_DEFAULT_REGION="eu-central-1"
	export AWS_INSTANCE_TYPE="t2.micro"
	```

 2. Create Docker instances:
	```bash
	# Define nodes names
	SWARM_MANAGER_NAME="Swarm-Manager"
	SWARM_NODE_NAME="Swarm-Node"
	
	
	# Create Swarm Manager
	docker-machine create --driver amazonec2 --amazonec2-open-port "2377" --amazonec2-open-port "7946" --amazonec2-open-port "7946"/udp "$SWARM_MANAGER_NAME"
	
	
	# Create Swarm Node
	docker-machine create --driver amazonec2 "$SWARM_NODE_NAME"
	```


 3. Configure first instance as Docker Swarm manager:
	```bash
	# Get Swarm manager IP
	SWARM_MANAGER_IP="$(docker-machine ssh "$SWARM_MANAGER_NAME" curl http://169.254.169.254/latest/meta-data/local-ipv4)"
	
	# Initialize cluster manager
	docker-machine ssh "$SWARM_MANAGER_NAME" sudo docker swarm init --advertise-addr "$SWARM_MANAGER_IP"
	
	# Get Swarm cluster token
	TOKEN="$(docker-machine ssh "$SWARM_MANAGER_NAME" sudo docker swarm join-token worker --quiet)"
	```

 4. Configure second instance as Docker Swarm cluster nodes:
	```bash
	# Add node to the cluser using output from the step above
	docker-machine ssh "$SWARM_NODE_NAME" sudo docker swarm join --token "$TOKEN" "$SWARM_MANAGER_IP":2377
	```


## Verification

 1. View swarm cluster nodes:
	```bash
	docker-machine ssh "$SWARM_MANAGER_NAME" sudo docker node ls
	```


## Remove environment

 1. Remove Docker Swarm manager and node instances:
	```bash
	docker-machine rm --force "$SWARM_MANAGER_NAME" "$SWARM_NODE_NAME"
	```


## Known issues

 1. Sometimes, docker-machine may fail to create a docker instance on AWS. In such cases failed instance should be removed and then recreated. For removing, please see [Clean envorinment](#clean-envorinment).
