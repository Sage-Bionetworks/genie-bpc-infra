# genie-bpc-infra

CDK-based script for deploying the Genie-BPC containerized application to AWS

`HOSTED_ZONE_NAME` and `HOSTED_ZONE_ID` in `.github/workflows/docker_deploy.yml' should match a Zone created in Route53 in the target AWS account.


The target account should have a secret in AWS Secrets Manager whose name matches the value of `STACK_NAME_PREFIX` in `docker_deploy.yml`,
and containing key-value pairs for `OAUTH_CLIENT_ID` and `OAUTH_CLIENT_SECRET`.


`role-to-assume` in `.github/workflows/docker_deploy.yml` controls which account the application is deployed into. The target account should have a 
policy allowing a role to assume the roles created by CDK during the (one-time) account initialization.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AssumeRoleStatement",
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::449435941126:role/cdk-*-role-*-us-east-1"
        }
    ]
}
```

and the role should have this policy in its permissions.  The role should also have the trust relationship 
to allow GitHub to deploy to the account:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::<account-id>:oidc-provider/token.actions.githubusercontent.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
                    "token.actions.githubusercontent.com:sub": [
                        "repo:Sage-Bionetworks/genie-bpc-infra:ref:refs/heads/main"
                    ]
                }
            }
        }
    ]
}
```
