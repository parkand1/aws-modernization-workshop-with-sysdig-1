---
title: "Configure workspace for Sysdig Workshop"
chapter: false
weight: 23
---

{{% notice info %}}
Cloud9 normally manages IAM credentials dynamically. This isn't currently compatible with
the EKS IAM authentication, so we will disable it and rely on the IAM role instead.
{{% /notice %}}

  1. Return to your workspace and click the gear icon (in top right corner), or click to open a new tab and choose "Open Preferences"

  2. Select **AWS SETTINGS** and turn off **AWS managed temporary credentials**

  3. Close the Preferences tab

     <img src=/images/10_prerequisites/iamRoleWorkspace.gif width="100%" >

  4. Copy paste the following two commands to configure your __Secure API Token__ and  __Secure API Endpoint__ environment variables (these are the values you made a note of [here]({{< ref "/10_prerequisites/11_sysdig.md" >}}))

```sh
echo "Enter your **Sysdig Secure API Token**"; read SecureAPIToken 
export SecureAPIToken 
```

```sh
echo "Enter your **Sysdig Secure API Endpoint**"; read SecureEndpoint
export SecureEndpoint
```

  5. And copy and run (paste with **Ctrl+P**) the command below. Before running it, review what it does at the end of this step.


```sh
sudo yum -y install jq
rm -vf ${HOME}/.aws/credentials
export ACCOUNT_ID=$(aws sts get-caller-identity --output text --query Account)
export AWS_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')
test -n "$AWS_REGION" && echo AWS_REGION is "$AWS_REGION" || echo AWS_REGION is not set
echo "export ACCOUNT_ID=${ACCOUNT_ID}" | tee -a ~/.bash_profile
echo "export AWS_REGION=${AWS_REGION}" |
tee -a ~/.bash_profile
aws configure set default.region ${AWS_REGION}
aws configure get default.region
curl -s https://gist.githubusercontent.com/johnfitzpatrick/d55097212d9bb4e1442383a5e3339b01/raw/90aa0dbb5b7e35277aea87fad12879e987f4c820/deploy-amazon-ecs-sample.sh > deploy-amazon-ecs-sample.sh
chmod +x deploy-amazon-ecs-sample.sh
curl -s https://gist.githubusercontent.com/johnfitzpatrick/95ecc8853764785a6033fd920ada1b3a/raw/92ff710c1f0622210b4a9917d2f775089f193073/ecs-cluster-env-vars.sh > ecs-cluster-env-vars.sh
chmod +x ecs-cluster-env-vars.sh
aws sts get-caller-identity --query Arn | grep Sysdig-Workshop-Admin -q && echo "IAM role valid" || echo "IAM role NOT valid"
```
{{% notice warning %}}
If the IAM role is not valid, <span style="color: red;">**DO NOT PROCEED**</span>. Go back and confirm the steps on this page.
{{% /notice %}}

### Explanation of the commands:

Actions executed:
:small_blue_diamond: Install jq - jq is a command-line tool for parsing JSON

:small_blue_diamond: Ensure temporary credentials aren’t already in place.

:small_blue_diamond: Remove any existing credentials file.

:small_blue_diamond: Set the region to work with our desired region.

:small_blue_diamond: Validate that our IAM role is valid.

:small_blue_diamond: Copy two scripts into place for use later in the workshop.
