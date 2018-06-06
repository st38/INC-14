# Task INC-14

 1. [Goals](#goals)
 2. [Description](#description)
 3. [Requirements](#requirements)
 4. [Duration](#duration)
 5. [General plan](#general-plan)
 6. [Usage](#usage)
 7. [Clean envorinment](#clean-envorinment)
 8. [Known issues](#known-issues)


## Goals

 Setup small docker swarm cluster


## Description

 Set up cluster consisting of 1 master and 1 worker (t2.micro). This cluster will be used for additional deployment tasks.


## Requirements

 1. A Linux host with Docker Machine installed.
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

 1. Provide AWS credentials:
	```bash
	export AWS_ACCESS_KEY_ID=access key
	export AWS_SECRET_ACCESS_KEY=secret access key
	```

 2. Create Docker instances:
	```bash
	REGION="eu-central-1"
	SWARM_MANAGER_NAME="Swarm-Manager"
	SWARM_NODE_NAME="Swarm-Node"
	INSTANCE_TYPE="t2.micro"
	DOCKER_PORT="2376"
	SWARM_PORT="2377"
	NODES_PORT="7946"
	OVERLAY_PORT="4789"
	
	
	# Create Swarm Manager
	docker-machine create --driver amazonec2 --amazonec2-region "$REGION" --amazonec2-instance-type "$INSTANCE_TYPE" "$SWARM_MANAGER_NAME"
	
	
	# Create Swarm Node
	docker-machine create --driver amazonec2 --amazonec2-region "$REGION" --amazonec2-instance-type "$INSTANCE_TYPE" --amazonec2-open-port "$SWARM_PORT" --amazonec2-open-port "$NODES_PORT" --amazonec2-open-port "$NODES_PORT"/udp --amazonec2-open-port "$OVERLAY_PORT"/udp "$SWARM_NODE_NAME"
	```


### Configure first instance as Docker Swarm manager

 1. Run the following commands:
	```bash
	# Make connection to the Swarm manager as active
	eval $(docker-machine env "$SWARM_MANAGER_NAME")
	
	# Get Swarm manager IP
	SWARM_MANAGER_IP="$(docker-machine ssh "$SWARM_MANAGER_NAME" curl http://169.254.169.254/latest/meta-data/local-ipv4)"
	
	# Initialize cluster manager
	docker swarm init --advertise-addr "$SWARM_MANAGER_IP"
	
	# Get Swarm cluster token
	TOKEN="$(docker swarm join-token worker --quiet)"
	```


### Configure second instance as Docker Swarm cluster nodes

 1. Run the following commands:
	```bash
	# Make connection to the Swarm node as active
	eval $(docker-machine env "$SWARM_NODE_NAME")
	
	# Add node to the cluser using output from the step above
	docker swarm join --token "$TOKEN" "$SWARM_MANAGER_IP":2377
	```


### Verify cluster status

 1. Run the following command:
	```bash
	# Make connection to the Swarm manager as active
	eval $(docker-machine env "$SWARM_MANAGER_NAME")
	
	# View swarm nodes
	docker node ls
	```


## Clean envorinment

 1. Remove Docker Swarm manager and node instances:
	```bash
	docker-machine rm --force "$SWARM_MANAGER_NAME" "$SWARM_NODE_NAME"
	```


## Known issues

 1. Sometimes, docker-machine may fail to create a docker instance on AWS. In such cases failed instance should be removed and then recreated. For removing, please see [Clean envorinment](#clean-envorinment).
 2. When docker-machine creates an instance and we specify the ports for security group [it may fail if we open ports twice](https://blog.berkgokden.com/creating-docker-engine-swarm-mode-cluster-in-amazon-ec2-with-docker-machine-docker-aws-8b46cf1e12a5).