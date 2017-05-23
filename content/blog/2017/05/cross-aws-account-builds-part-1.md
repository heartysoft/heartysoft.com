+++
title = "Cross AWS Account Builds - Part 1"
description = "What you need to have build servers accessing stuff in another AWS Account."
tags = [ "blog", "devops", "aws" ]
date = "2017-05-23 18:27:05.00"
slug = "cross-aws-account-builds-part-1"
+++

Getting cross AWS account permissioning working seems like a black art. At my current client, we have our own AWS Account, with stuff like Kubernetes, Elasticsearch, Postgres, Cassandra, etc. running. However, another team manages a Gitlab Enterprise set up which is hosting our codebases. We *could* set up our own source control, and build server, and we very well might do that. But for now, the quickest bang for the buck is to leverage what the other team are allowing us to use. The catch is - they're running on a separate AWS Account. 

Let's say our AWS Account is called OURTHING, and the build server's environment is called BUILD. I won't go through all the automation involved, but will outline steps that can get to where we want. 

## 1. Create a role in OURTHING

The first step is to create a Role for Cross Account Access, ideally with an External Id. The result of this can be a role that looks like the following:

```
{
    "Role": {
        "Path": "/",
        "RoleName": "OurThingBuildRole",
        "RoleId": "ROLE_ID",
        "Arn": "arn:aws:iam::ACCOUNT:role/OurThingBuildRole",
        "CreateDate": "2017-05-23T14:50:09Z",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "AWS": "arn:aws:iam::BUILDACCOUNT:root"
                    },
                    "Action": "sts:AssumeRole",
                    "Condition": {
                        "StringEquals": {
                            "sts:ExternalId": "some_key_build"
                        }
                    }
                }
            ]
        },
        "Description": "Role for build server to do stuff with our stuff"
    }
}

```

Notice the use of BUILDACCOUNT in the policy statement. This needs to be the account that the build server will be running in. Take note of the Role Arn for OurThingBuildRole; we will need this later. 

## 2. Give the role the required policies to access the resources in OURTHING

This would depend on what the build needs to do. For our case, I just want the build server to be able to push docker container images to ECR. As such, I've just attached the AmazonEC2ContainerRegistryPowerUser managed AWS policy to the OurThingBuildRole.

With that in place, we're ready to flip over to the other side; we will do that in part 2.


