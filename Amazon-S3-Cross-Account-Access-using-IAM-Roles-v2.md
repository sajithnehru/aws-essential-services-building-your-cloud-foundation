## Cross-Account S3 Bucket Access Using IAM Roles
**Scenario:**   
*S3 Bucket:* spndemo-crossaccount-role-bucket2 in Account A  
*IAM User:* accountb-user in Account B  
*Region:* us-east-1  
*Objective:* Grant the user in Account B access to the S3 bucket in Account A by configuring IAM roles.  

**Step 1:**  
Test Access Before Applying IAM roles  

Create AWS Profile for Testing Access  
- Log in to Account B.  
- Navigate to IAM > Users > accountb-user > Security Credentials > Create Access Key.  
- Copy the Access Key ID and Secret Access Key.


Open your command line interface and configure AWS CLI with the following command:  
```
aws configure --profile account-b
```

Provide the Access Key ID, Secret Access Key, and Region (us-east-1).

Test S3 Access
Run the following command to test access to the S3 bucket:

```
aws s3 ls s3://spndemo-crossaccount-role-bucket2 --profile account-b  
```

You should receive an "Access Denied" error because no policies have been applied yet.

**Step 2:** 
Configure IAM Role in Account A  

Create Role:   
- Log in to Account A.  
- Navigate to IAM > Roles > Create Role > AWS Account > Another Account.  
- Enter the Account ID of Account B.  
- Click Next and proceed with the default settings.  
- Enter a role name, e.g., cross_account_access_accounta.  
- Click Create Role.  

Update Trust Relationships in Account A:  

Copy the ARN of the user in Account B:  
- Account B > IAM > Users > accountb-user > Copy ARN: arn:aws:iam::185408969973:user/accountb-user.  
Update Trust Policy in Account A:  
- Account A > IAM > Roles > cross_account_access_accounta > Trust Relationships > Edit Trust Policy.
- Replace arn:aws:iam::185408969973:root with arn:aws:iam::185408969973:user/accountb-user.

Add S3 List Permission to the IAM Role:   
- Account A > IAM > Roles > cross_account_access_accounta > Permissions > Add permissions > Create inline policy.  
Use the visual editor to create the policy:  
- Service: S3
- Action: List > List Buckets
- Resource: Add ARN: arn:aws:s3:::spndemo-crossaccount-role-bucket2
- Click Review policy and then Create policy.

**Step 3:**
Add Assume Role Permission in Account B  
- Account B > IAM > Users > accountb-user > Permissions > Create inline policy.
Use the visual editor to create the policy:  
- Service: STS
- Action: AssumeRole
- Resource: Add ARN: arn:aws:iam::554294332649:role/cross_account_access_accounta
- Click Review policy and then Create policy.

**Step 4:**
Assume the Role and Test Access  

Assume the Role:  

Run the following command to assume the role:

```
aws sts assume-role --role-arn arn:aws:iam::554294332649:role/cross_account_access_accounta --role-session-name cross-account-session-v2 --profile account-b --output json  
```

The above command will output the required credentials (AccessKeyId, SecretAccessKey, SessionToken).

Configure a New AWS Profile
Configure a new AWS profile with the obtained credentials:

```
aws configure --profile sessions  
```

Provide the AccessKeyId, SecretAccessKey, and region (us-east-1).

Set the session token:
```
aws configure set aws_session_token IQoJb3Jp.............pwjnbmocC48= --profile sessions
```

Test S3 Access
Run the following command to list the objects in the S3 bucket:
```
aws s3 ls s3://spndemo-crossaccount-role-bucket2 --profile sessions
```

If the configuration is correct, the command should list the objects in the S3 bucket, indicating successful cross-account access.

That's it!!!  

** I reqeust you to replace the following when you use this script.  

Account A - S3 bucket name  
Account B - IAM username  
ARN of IAM user and s3 bucket  
Account A - IAM role name  
Region name  
