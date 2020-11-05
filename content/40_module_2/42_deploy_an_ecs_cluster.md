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

    We'll use a custom deployment script that will

  - Retrieve the id of the default security group for the VPC created, and allows inbound access on port 80

  - Create a `ecs-params.yml` file using the subnets and security group already retrieved

  - Create a `docker-compose.yaml` to instantiate the image `amazon/amazon-ecs-sample`

3. Run the script `deploy-amazon-ecs-sample.sh`, copying and pasting the VPC & Subnet values from the above out when prompted

    ```bash
    cd /home/ec2-user/environment
    ./deploy-amazon-ecs-sample.sh
    ```

    **Note** You can subsequently get the VPC and Subnet details requested from the [CloudFormation UI](https://console.aws.amazon.com/cloudformation/home)

    ![ECS Cluster](/images/40_module_2/image7.png)


    Optionally, for details of this script you can run the following command

    ```bash
    cat ./deploy-amazon-ecs-sample.sh
    ```

6. Once completed you can see on the [Amazon ECS UI](https://console.aws.amazon.com/ecs/home?region=us-east-1#/clusters/tutorial/services)

![Cluster Tutorial](/images/40_module_2/image5.png)
