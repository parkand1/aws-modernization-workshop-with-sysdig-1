---
title: "Deploy an ECS cluster using Fargate"
chapter: false
weight: 42
---

To illustrate automatic scanning, we will now deploy a sample ECS cluster that scales using Fargate.  For the purposes of the lab this will consist of this sample PHP appliction running in a Docker Compose environment - https://hub.docker.com/r/amazon/amazon-ecs-sample.

1. Create a cluster configuration and create a cluster

    ```
    ecs-cli configure --cluster tutorial --default-launch-type FARGATE --config-name tutorial --region us-east-1

    ecs-cli up --cluster-config tutorial --ecs-profile tutorial-profile
    ```

    The output should show a VPC and two Subnets have been created:-

    ```
    INFO[0000] Created cluster                    cluster=tutorial region=us-east-1
    INFO[0000] Waiting for your cluster resources to be created...
    INFO[0000] Cloudformation stack status       stackStatus=CREATE_IN_PROGRESS
    INFO[0060] Cloudformation stack status       stackStatus=CREATE_IN_PROGRESS
    VPC created: vpc-046ed77edcd796e19
    Subnet created: subnet-045df8f58a51b2291
    Subnet created: subnet-0e4623283c4907ea7
    Cluster creation succeeded.
    ```


1. Set the names of the VPC and Subnets **from the output above** as an environment variables, as follows

    ```bash
    export VPC=vpc-046ed77edcd796e19
    export SUBNET1=subnet-045df8f58a51b2291
    export SUBNET2=subnet-0e4623283c4907ea7
    ```

    **Note** You can subsequently get these details from the [CloudFormation UI](https://console.aws.amazon.com/cloudformation/home)


    ![ECS Cluster](/images/40_module_2/image7.png)

    We'll use a custom deployment script that will

  - Retrieve the id of the default security group for the VPC creates, and allows inbound access on port 80

  - Create a `ecs-params.yml` file using the subnets and security group already retrieved

  - Create a `docker-compose.yaml` to instantiate the image `amazon/amazon-ecs-sample`

3. Run the script `deploy-amazon-ecs-sample.sh`

    ```bash
    cd /home/ec2-user/environment
    ./deploy-amazon-ecs-sample.sh
    ```

    <details>
    <summary>Expand for details of script `deploy-amazon-ecs-sample.sh`</summary>
      ```
      1  export group_id=$(aws ec2 describe-security-groups --filters Name=vpc-id,Values=$VPC --region us-east-1 | jq '.SecurityGroups[0].GroupId' | xargs)
      2
      3  aws ec2 authorize-security-group-ingress --group-id $group_id --protocol tcp --port 80 --cidr 0.0.0.0/0 --region us-east-1
      4
      5  cat <<- 'EOF' > "docker-compose.yml"
      6  version: '3'
      7  services:
      8    web:
      9      image: amazon/amazon-ecs-sample
     10      ports:
     11        - "80:80"
     12      logging:
     13        driver: awslogs
     14        options:
     15          awslogs-group: tutorial
     16          awslogs-region: us-east-1
     17          awslogs-stream-prefix: web
     18  EOF
     19
     20  cat <<- 'EOF' > "ecs-params.yml"
     21  version: 1
     22  task_definition:
     23    task_execution_role: ecsTaskExecutionRole
     24    ecs_network_mode: awsvpc
     25    task_size:
     26      mem_limit: 0.5GB
     27      cpu_limit: 256
     28  run_params:
     29    network_configuration:
     30      awsvpc_configuration:
     31        subnets:
     32          - "subnet1"
     33          - "subnet2"
     34        security_groups:
     35          - "sg1"
     36        assign_public_ip: ENABLED
     37  EOF
     38
     39  sed -i "s/subnet1/$SUBNET1/g" ecs-params.yml
     40  sed -i "s/subnet2/$SUBNET2/g" ecs-params.yml
     41  sed -i "s/sg1/$group_id/g" ecs-params.yml
     42
     43  ecs-cli compose --project-name tutorial service up --create-log-groups --cluster-config tutorial --ecs-profile tutorial-profile
     ```
    </details>


6. Once completed you can see on the [Amazon ECS UI](https://console.aws.amazon.com/ecs/home?region=us-east-1#/clusters/tutorial/services)

![Cluster Tutorial](/images/40_module_2/image5.png)
