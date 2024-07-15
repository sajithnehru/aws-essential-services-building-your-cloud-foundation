**Cross-Account S3 Access Using Bucket and IAM Policies**  

**Scenario**  
You have:

An S3 bucket (spndemo-crossaccount-source-bucket-a) in Account A  
An IAM user (crossaccount-userb) in Account B  
Region: us-east-1  
Objective: Grant the user in Account B access to the S3 bucket in Account A by configuring bucket and IAM policies.  

**Step 1: Test Access Before Applying Policies**  
Create AWS Profile for Testing Access

- Log in to Account B.
- Navigate to IAM > Users > crossaccount-userb > Security Credentials > Create Access Key.
- Copy the Access Key ID and Secret Access Key.
- Open your command line interface and configure AWS CLI with the following command:
```
aws configure --profile accountb
```  
Provide the Access Key ID, Secret Access Key, and Region (us-east-1).  
Test S3 Access
Run the following command to test access to the S3 bucket:

```
aws s3 ls s3://spndemo-crossaccount-source-bucket-a --profile accountb
```
You should receive an "Access Denied" error because no policies have been applied yet.  

**Step 2: Configure S3 Bucket Policy in Account A**  
Create and Attach Bucket Policy  

- Log in to Account A.
- Go to Amazon S3 > Buckets > spndemo-crossaccount-source-bucket-a > Permissions > Bucket Policy > Edit > Policy Generator.
- Configure the policy:
- Type of Policy: S3 Bucket Policy
- Principal: ARN of the user in Account B (arn:aws:iam::185408969973:user/crossaccount-userb)
- Actions: Select All Actions
- Amazon Resource Name (ARN): arn:aws:s3:::spndemo-crossaccount-source-bucket-a
- Click "Add Statement."  

Add Additional Policy

Repeat the above steps but change the ARN to arn:aws:s3:::spndemo-crossaccount-source-bucket-a/*.

- Click "Generate Policy" and copy the policy JSON.  
- Paste the JSON into the bucket policy editor and save the changes.

**Step 3: Create IAM Policy for User in Account B**  
Create and Attach IAM Policy

- Log in to Account B.
- Go to IAM > Policies > Create Policy > Policy Generator.

Configure the policy:

- Type of Policy: IAM Policy
- Principal: ARN of the user in Account B (arn:aws:iam::185408969973:user/crossaccount-userb)
A- WS Service: Amazon S3
- Actions: Select All Actions
- Amazon Resource Name (ARN): arn:aws:s3:::spndemo-crossaccount-source-bucket-a
- Click "Add Statement."
- Generate the policy JSON.
- Attach Policy to IAM User

Go to IAM > Users > crossaccount-userb > Permissions > Add Permissions > Create Inline Policy > JSON.
Paste the generated policy JSON and save it with a name (e.g., Cross-account-iam-policy).  

**Step 4: Test Access After Applying Policies**  
Test S3 Access Again  

Open your command line interface and run:
```
aws s3 ls s3://spndemo-crossaccount-source-bucket-a --profile accountb
```
The command should now list the objects in the S3 bucket, indicating successful cross-account access.  

That's it!!!  

You have successfully set up cross-account S3 access using bucket and IAM policies.

Disclaimer:
I reqeust you to replace the following when you use this script. 

Account A - S3 bucket name  
Account B - IAM username  
ARN of IAM user and s3 bucket  
Region name  
