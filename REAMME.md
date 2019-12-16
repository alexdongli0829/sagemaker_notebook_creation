1. create policy for the policy for the notebook
https://docs.aws.amazon.com/glue/latest/dg/create-sagemaker-notebook-policy.html

aws iam create-policy --policy-name glue_notebook_used --policy-document file://role_policy.json

{
    "Policy": {
        "PolicyName": "glue_notebook_used",
        "PolicyId": "ANPASMGISPI4THJTKAZ5S",
        "Arn": "arn:aws:iam::163629398585:policy/glue_notebook_used",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2019-12-15T23:49:16Z",
        "UpdateDate": "2019-12-15T23:49:16Z"
    }
}


2. Create the role for the notebook

https://docs.aws.amazon.com/glue/latest/dg/create-an-iam-role-notebook.html?icmpid=docs_glue_console

#"AWSGlueServiceNotebookRole-" should be included in the name

aws iam create-role --role-name AWSGlueServiceNotebookRole-cliversion --assume-role-policy-document file://assume_role_policy.json


{
    "Role": {
        "Path": "/",
        "RoleName": "AWSGlueServiceNotebookRole-cliversion",
        "RoleId": "AROASMGISPI4XI7UUAPKK",
        "Arn": "arn:aws:iam::163629398585:role/AWSGlueServiceNotebookRole-cliversion",
        "CreateDate": "2019-12-15T23:51:00Z",
        "AssumeRolePolicyDocument": {
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "Service": "sagemaker.amazonaws.com"
                    },
                    "Action": "sts:AssumeRole"
                }
            ],
            "Version": "2012-10-17"
        }
    }
}


3. Attach the role with the policy

aws iam attach-role-policy --role-name AWSGlueServiceNotebookRole-cliversion --policy-arn arn:aws:iam::163629398585:policy/glue_notebook_used



4. Create Glue endpoint

aws glue create-dev-endpoint --endpoint-name sgnotebook_endpoint --role-arn arn:aws:iam::163629398585:role/GlueWithAllS3GetForTutorial --security-group-ids sg-02e66f98d412aca4c --subnet-id subnet-8128b0f6 --number-of-nodes 3 --public-key "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCbsbgru4GmUE3OVpDq6np+9KNtmjEHmTKwswI05/+EQtTo4tYOT3L9wafpOBGx3DJH+MK5DO0PVisCuSTQKrowRmqk/6IdtH0ZaSQ6UK/+HwPHhdTay1M2OnMihjm4ccLnpwwPk0cyBPDSH2uT1cBbSfFpiRUIAADPgK+8KSbHtuwaskp/sA5cOSWiIEKsStuESvCakDWMEaRznUUheGK5J4FOSpKo9w3OsAscoOY3ksf0e6OOUBtgTl2PENSIHTy0lSrKbG8jzNFAEDM5cimjPvcXvO/Xcx063zoR9Z8wMU28mm4MhO7KW332whbwL2qe0LH0EWMO/XN41gFN9k+h dongaws-sdy"


{
    "EndpointName": "sgnotebook_endpoint",
    "Status": "PROVISIONING",
    "SecurityGroupIds": [
        "sg-02e66f98d412aca4c"
    ],
    "SubnetId": "subnet-8128b0f6",
    "RoleArn": "arn:aws:iam::163629398585:role/GlueWithAllS3GetForTutorial",
    "ZeppelinRemoteSparkInterpreterPort": 0,
    "NumberOfNodes": 5,
    "AvailabilityZone": "ap-southeast-2a",
    "VpcId": "vpc-74c15511",
    "CreatedTimestamp": 1576449062.298
}


5. Create liftcycle configuration

aws sagemaker create-notebook-instance-lifecycle-config --notebook-instance-lifecycle-config-name glue-endpoint-config --on-start Content=$((cat start_notebook.sh || echo "")| base64) --on-create Content=$((cat start_notebook.sh || echo "")| base64)

{
    "NotebookInstanceLifecycleConfigArn": "arn:aws:sagemaker:ap-southeast-2:163629398585:notebook-instance-lifecycle-config/glue-endpoint-config"
}


6. Create sagemaker notebook instance

# note, the name should include "aws-glue-" which will make the notebook show in the glue console


aws sagemaker create-notebook-instance --notebook-instance-name aws-glue-cli-test2 --instance-type ml.t2.medium --subnet-id subnet-8128b0f6 --security-group-ids sg-02e66f98d412aca4c --role-arn arn:aws:iam::163629398585:role/AWSGlueServiceNotebookRole-cliversion --tags Key=aws-glue-dev-endpoint,Value=sgnotebook_endpoint Key=aws-glue-notebook-status,Value=created --lifecycle-config-name glue-endpoint-config
