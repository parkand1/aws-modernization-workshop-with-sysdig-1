---
title: "Detecting Runtime Cloud Security Threats"
chapter: false
weight: 52
---

Let's look at an example of AWS threat detection in action with CloudTrail and the Sysdig CloudConnector.  To do so we'll create an S3 bucket, and make it public

1. Log into Cloud9 Workspace
2. Create an S3 bucket. S3 bucketnames are globally unique, so **use your initials** combined with a timestamp

    ```
    INITIALS=<your initial>
    BUCKETNAME=$INITIALS-$(date +%s)
    aws s3api create-bucket --bucket $BUCKETNAME --acl public-read
    ```

3. Now delete the S3 bucket's encryption.  This should be considered a potential security threat.

    ```
    aws s3api delete-bucket-encryption --bucket $BUCKETNAME
    ```

4. To view details of this event, browse to [CloudTrail](https://console.aws.amazon.com/cloudtrail/home) then 'Event History'

    ![CloudTrail](/images/50_module_3/cloudtrail01.png)

    If you scroll down you'll see details of the new CloudTrail event in JSON format:

5. Below is an example of a 'DeleteBucketEncryption' event raised after our previous command

{{% notice info %}}
It can take several minutes for new events to appear in CloudTrail. In the meantime you can browse the existing events created earlier from earlier activity in the account.
{{% /notice %}}

```JSON
{
"eventVersion": "1.05",
"userIdentity": {
    "type": "AssumedRole",
    "principalId": "AROJ2JML2BN4IFASO2OGL:MasterKey",
    "arn": "arn:aws:sts::161134881107:assumed-role/TeamRole/MasterKey",
    "accountId": "161134881107",
    "accessKeyId": "ASIAJRUKKVSOVIBJRNLA",
    "sessionContext": {
        "sessionIssuer": {
            "type": "Role",
            "principalId": "AROJ2JML2BN4IFASO2OGL",
            "arn": "arn:aws:iam::161134881107:role/TeamRole",
            "accountId": "161134881107",
            "userName": "TeamRole"
        },
        "webIdFederationData": {},
        "attributes": {
            "mfaAuthenticated": "false",
            "creationDate": "2020-10-19T15:06:44Z"
        }
    },
    "invokedBy": "cloudformation.amazonaws.com"
},
"eventTime": "2020-10-19T17:24:16Z",
"eventSource": "kms.amazonaws.com",
"eventName": "ScheduleKeyDeletion",
"awsRegion": "us-east-1",
"sourceIPAddress": "cloudformation.amazonaws.com",
"userAgent": "cloudformation.amazonaws.com",
"requestParameters": {
    "keyId": "d3914791-8d06-47b5-a976-2fae1851a848"
},
"responseElements": {
    "keyId": "arn:aws:kms:us-east-1:168110711348:key/d4739191-8d06-47b5-a976-21a8fe1854a8",
    "deletionDate": "Nov 19, 2020 12:00:00 AM"
},
"requestID": "a9667575-e5b2-488a-b2bf-a3c335887d51",
"eventID": "78c15027-2ad8-492e-a26f-058a0ce48e96",
"readOnly": false,
"resources": [
    {
        "accountId": "161134881107",
        "type": "AWS::KMS::Key",
        "ARN": "arn:aws:kms:us-east-1:161134881107:key/d4739191-8d06-47b5-a976-21a8fe1854a8"
    }
],
"eventType": "AwsApiCall",
"recipientAccountId": "161134881107"
}
```

{{% notice info %}}
Please note that all data in the JSON doc above is fictitious.
{{% /notice %}}


All CloudTrail events have the following key fields:

- **userIdentity**: The user who sent the request.
- **eventName**: Specifies the type of event.
- **requestParameters**: Contains all of the parameters related to the request.

If a request has an **errorCode** field, it means that it could not be processed because of an error. For example, the requester may not have had permission to perform a change.

In this case, we can see how a policy has just been attached (**AttachUserPolicy**) to a user (**admin_test**) with administrator access (**arn:aws:iam::aws:policy/AdministratorAccess**).

A Falco rule to detect this elevation of privileges would look like this:


```
- rule: Delete bucket encryption
  desc: Detect deleting configuration to use encryption for bucket storage
  condition:
    jevt.value[/eventName]="DeleteBucketEncryption" and not jevt.value[/errorCode] exists
  output:
    A encryption configuration for a bucket has been deleted
    (requesting user=%jevt.value[/userIdentity/arn],
     requesting IP=%jevt.value[/sourceIPAddress],
     AWS region=%jevt.value[/awsRegion],
     bucket=%jevt.value[/requestParameters/bucketName])
  priority: CRITICAL
  tags: [cloud, source=cloudtrail, aws, NIST800_53, NIST800_53_AU8]
  source: k8s_audit
```

Some points to note about this rule:

 - The **jevt.value** contains the JSON content of the event, and we are using it in the **condition**. Using the [jsonpath](https://jsonpath.com/) format, we can indicate what parts of the event we want to evaluate.

 - The **output** will provide context information including the requester **username** and **IP address** - this is what will be sent through all of the enabled notification channels.

As you can see, this is a regular Falco rule. In fact, this particular rule is already included out-of-the-box in Sysdig CloudConnector. CloudTrail compatibility is achieved by handling its events as JSON objects, and referring to the event information using JSONPath.
