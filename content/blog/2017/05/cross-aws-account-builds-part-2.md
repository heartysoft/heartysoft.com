+++
title = "Cross AWS Account Builds - Part 2"
description = "What you need to have build servers accessing stuff in another AWS Account."
tags = [ "blog", "devops", "aws" ]
date = "2017-05-29 16:05:05.00"
slug = "cross-aws-account-builds-part-2"
+++

At the end of [part 1](http://heartysoft.com/ashic/blog/2017/05/cross-aws-account-builds-part-1/), we have an IAM role created in our account, OURTHING which has the required permissions in our account, and allows a user in the BUILD account to assume this role provided they specify a matching ExternalId - which in this case is "some_key_build". 

## Steps for BUILDACCOUNT

We'll need an IAM user in BUILD account which we can use to run our builds. We'll need the BUILD account admin to create this for us. They can then give us the user's AWS_ACCESS_KEY_ID, and AWS_SECRET_ACCESS_KEY. It's worth noting that we never use these credentials from our account. In fact, if the build server running on BUILD account sets these environment variables, then we don't need to know the values at all. 

In addition to creating the account, the user will also need a policy applied to it that will enable it to assume the role we created on our side. The policy to apply will look like:

```
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "sts:AssumeRole",
    "Resource": "arn:aws:iam::OURACCONT:role/OURTHINGBUILDROLE"
  }]
}
```

Note, the resource is the ARN of the AWS IAM Role we created under our account in part 1.

## Assuming the role - way 1

Before running our build, we'll need to assume the role we've set up. This can be done in two ways. We can run

```
 aws sts assume-role --role-arn arn:aws:iam::OURACCOUNT:role/OURTHINGBUILDROLE --role-session-name something
```

This will return a json document with a Credentials property, which has fields like SecreteAccessKey, AccessKeyId, etc. You can extract the json values, and set environment variables. You'd then be able to access what you need. For example:

```
$(aws ecr get-login)
```

## Assuming the role - way 2

Another way to assume the role is through profiles. On the build server, you can create a file at ~/.aws/config that looks like:

```
[profile build]
role_arn=OURBUILDTHINGROLE
external_id=<the ExternalId used in part 1>
source_profile=build
region=eu-west-1
```

and a corresponding ~/.aws/credentials file:

```
[build]
aws_access_key_id=<access key id for AWS IAM user running the build>
aws_secret_access_key=<that user's key>
```

Then, we can do what we need by specifying the profile:

```
$(aws ecr get-login --profile build)
```

I'm using docker containers to execute builds, and have a simple script that sets up the profiles at [configure-aws.sh](https://github.com/heartysoft/docker-builder-aws). The second approach works fine for me in this environment.

## Summary

In short, the step we need to take are:

- Create a role in our account, grant it the permissions it needs.
- Have the BUILD account create a user, and grant it assume role permissions for our role. 
- That user can then assume our role, allowing it to access our resource, push to our ECR repos, etc.

Of course, we could have simply created an IAM user in our account, and used its access keys from the build account. But this would require using long running keys in a different account. With this approach, a user's access keys never leaves its account. We can also add or remove the build role's capabilities from ourside, revoke sessions, and permissions without any help from the BUILD account once this is set up. 

