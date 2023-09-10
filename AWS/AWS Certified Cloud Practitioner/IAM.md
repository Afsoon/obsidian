**IAM** stands for Identy and Access Management Global service.
**Root account** created by default, shouldn't be used or shared, we should create **Users** and **Groups**.

**Groups only contain users**, not other groups.
**Users doesn't have to belong** to a group but **they can belong to multiples groups**. The first scenario is considered a bad practice.

![[Pasted image 20230904163137.png]]

We create users and groups to **manage in a sane way permissions**. Permissions are declared using JSON files named **policies**. An example of a policie
![[Pasted image 20230904163337.png]]
In AWS we apply the **least privilige principle**: don't give more permissions than a user needs.

## Policies

If a group has attached a policy, the users that belongs to the group inherit the policy. If a user doesn't belong to a group, we can create **inline policy**.

### Structure JSON policies
```json
{
	"Version": "2012-10-17",
	"Id": "S3-Account-Permissions",
	"Statement": [
		{
			"Sid": "1",
			"Effect": "Allow",
			"Principal": {
				"AWS": ["arn:aws:iam::"]
			},
			"Action": [
				"s3:GetObject",
				"s3:PutObject"
			],
			"Resource": ["arn:aws:<Service>:::"]
		}
	]
}
```
- **Version**: Stands for policy language version, **always include "2012-10-17"**.
- **Id**: The identifier, it's **optional**.
- **Statement**: An array of statements, it's **required**. Statements consist of
	- **Sid**: Statement identifier, it's **optional**.
	- **Effect**: The possible values are `Allow` or `Deny`
	- **Principal**: The account, user or role to which this policy applied to.
	- **Action**: List of actions this policy allows or denies.
	- **Resources**: List of resources to which the actions applied to.
	- **Condition**: Conditions for when this policy is in effect, it's **optional**.

## Password Policy

We can define a custom password policy, the options are:
- Length
- Character types
- Allow IAM to change their own password
- Require users to change their password after some time (password expiration)
- Prevent password re-use
- Activate MFA (Password you know and security device you own). Possible MFAs accepted
	- Virtual MFA: Authenticator apps like Google Authenticator. Supports for multiple tokens on a single device.
	- U2F (Universal 2nd Factor Security Key) Support for multiple root and IAM users using a single security key. Yubekiey
	- Hardware Key Fob MFA Device
	- Hardware Key Fob MFA Device for AWS GovCloud

## How to access AWS

We have 3 options:
- Management Console
- CLI: Protected by access keys
- SDK: For code

Access key are generated through the AWS Console
User manage their own access keys
**Access Keys are secret, just like a password. Don't share them**

**AWS Cloudshell** is a terminal on cloud but it is available in some regions.

## IAM Roles for Services

Some AWS service will need to perform actions on your behalf. To do so, we will assign **permissions** to AWS services with **IAM Roles**.

Common roles:

- EC2 Instance roles
- Lambda Function roles
- Roles for CloudFormation

**Roles** allow us to have credentials for a short duration and to do whatever they need to do. You can create roles for any services.

## Security Tools

**IAM Credentials Report (Account-level)**: a report that list all your account's users and the status of their various credentials

**IAM Access Advisor (user-level)**: show the service permissions granted to a user and when those services were last accessed.

We can use these tools to see what permissions are unused and remove it from the policies. 

## Best practices

- Never user your root account except for AWS account setup
- One physical user = One AWS user
- Assign users to groups and assign permission to groups
- Create a strong password policy
- Enforce MFA
- Create an use Roles for giving permission to AWS Services
- Use Access keys for Programmatic Access
- Audit permission of your account using IAM Credentials Report & IAM Access Advisor
- **Never share IAM users & Access Keys**

## Shared Responsibility Model for IAM

AWS (They are responsible for infrastructure)
- Infrastructure (global network security)
- Configuration and vulnerability analysis
- Compliance validation

You (You are responsible on how you use the infrastructure)
- Users, Groups, Roles, Policies management and monitoring
- enable MFA on all accounts
- Rotate all your keys often
- Use IAM tools to apply appropriate permissions
- Analyze access patterns & review permissions
