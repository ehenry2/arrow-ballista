<!---
  Licensed to the Apache Software Foundation (ASF) under one
  or more contributor license agreements.  See the NOTICE file
  distributed with this work for additional information
  regarding copyright ownership.  The ASF licenses this file
  to you under the Apache License, Version 2.0 (the
  "License"); you may not use this file except in compliance
  with the License.  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing,
  software distributed under the License is distributed on an
  "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
  KIND, either express or implied.  See the License for the
  specific language governing permissions and limitations
  under the License.
-->

# Deploying Ballista on AWS Fargate

Ballista can be deployed to AWS using the following instructions. These instructions assume you are already
comfortable deploying infrastructure to AWS.

The Ballista deployment consists of:

- A Fargate ECS cluster
- ECS service and task definition for one or more executor processes
- ECS service and task definition to the schedulers
- Elastic File System (EFS) volume to make local data accessible to Ballista
- Task IAM role
- Task Execution IAM role


## Build Docker Images

There are no officially published Docker images, so it is currently necessary to build the images from source.

Run the following commands to clone the source repository and build the Docker image.

```bash
git clone git@github.com:apache/arrow-ballista.git -b 0.11.0
cd arrow-ballista
./dev/build-ballista-docker.sh
```

This will create the following images:

- `apache/arrow-ballista-scheduler:0.11.0`
- `apache/arrow-ballista-executor:0.11.0`

## Publishing Docker Images

Once the images have been built, you can retag them and can push them to Amazon Elastic Container Registry (ECR) or your favourite registry.

```bash
# Retag
docker tag apache/arrow-ballista-scheduler:0.11.0 <aws_account_id>.dkr.ecr.<region>.amazonaws.com/<your-repo>/arrow-ballista-scheduler:0.11.0
docker tag apache/arrow-ballista-executor:0.11.0 <aws_account_id>.dkr.ecr.<region>.amazonaws.com/<your-repo>/arrow-ballista-executor:0.11.0

# Authenticate to ECR
aws ecr get-login-password --region <region> | docker login --username <user> --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com

# Push the docker images to the registry
docker push <aws_account_id>.dkr.ecr.<region>.amazonaws.com/<your-repo>/arrow-ballista-scheduler:0.11.0
docker push <aws_account_id>.dkr.ecr.<region>.amazonaws.com/<your-repo>/arrow-ballista-executor:0.11.0
```

## Publishing Docker Images