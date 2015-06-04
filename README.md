cloudformation-templates
==============

## Background

This group of CloudFormation templates is designed to allow you to quickly provision a new environment within AWS with support for the recommended security best-practices. For example, it sets up a logging bucket with all of the correct policies to allow AWS services to write to it. While I've worked hard to make these templates environment and user agnostic, there are still some places where you should customize the template before running it in your account. Additionally, these templates should be launched in order to avoid resource references that don't exist yet.

## Methodology

### New Accounts
These templates are designed to be used as-is with new accounts. If you are launching these into existing accounts that already have many resources, I suggest beginning new accounts and re-defining your infrastructure there. If that isn't possible, these templates can still be used, but with modifications.

### Separate Accounts for Each Environment
Although many organizations have multiple environments within one account, I highly recommend beginning a new account for each environment and tying them under a single billing account. This allows you to entirely isolate resouces and potential security issues.

### Mirror Image Accounts
The entire purpose of "infrastructure as code" is to script the setup and configuration of your infrastructure. By using these templates, you can create dev, qe, stage and prod environments that are mirror images of one another (an ideal setup). This can be done by simply launching the same template in each account and changing the "environment" parameter. Besides the organizational and code-reuse advantages, this also allows you to test infrastructure changes in dev, qe, and stage before finally launching in prod.

### Git and Infrastructure
To take full advantage of the "code" part of "infrastructure as code", I suggest creating an "infrastructure" Git repository with "dev", "qet", "stg", and "master" (or "prd") branches. Changes to your infrastructure can then work their way through this chain just like actual projects. When you push to these branches, either have Jenkins update the associated stacks that have changed or update them manually.

### Agnosticism and Reuseability
These templates have been designed with multiple users, environments, accounts, and setups in mind. The goal is to allow them to be launched with minimal changes in any environment and quickly deploy infrastructure. Feel free to fork this repository for your own organization's needs.

## Things to Keep in Mind

### Completeness
I've done my best to provide samples and coverage for most AWS services. However, this is a work in progress, so not everything is covered. For example, Lambda execution roles would require definitions in the IAM template. These have not been created yet. However, feel free to fork and modify as needed.

### Chicken and Egg
The S3 template defines a deployments bucket which will contain subfolders for all CloudFormation templates. However, in order to create that bucket, you have to provide a template. The solution is to upload the template JSON file first, allow it to create the bucket, then the next time an update happens, specify an S3 path instead.

## Common Parameters

Most of the templates prompt for a few parameters in order to help customize the environment.

* Owner: The owner should be your organization or user name. For example: "widgets" or "widgetco". This parameter should be lowercase with alphanumeric characters and dashes. It will be used in S3 bucket names and tags.
* Environment: This parameter allows the same stacks to be launched in different environments. Select from dev, qet (Quality Engineering and Test), stg, or prd.
* RootDomain: Some templates require the root domain of the organization. This is used in creating subdomains, CloudFront distributions, etc.

## Infrastructure Templates
The infrastructure templates define the base AWS resources used in an account. These should be setup first, before launching any projects. They define the underlying network setup, security groups, common S3 buckets, DNS names, and shared security policies.

### VPC
Coming soon (it's a big one)

### S3
The S3 template defines a series of buckets and storage resources used within the account. These include:

* Logging bucket: The logging bucket is setup as a centralized bucket for collection of logs from around the account. The necessary permissions are included on the bucket policy to allow AWS resources to save their logs to this bucket.
* AWS CloudTrail: CloudTrail is enabled and includes logs saved to the logging bucket.
* Sample public bucket: a sample bucket is provided that includes CORS headers for public access.
* Sample CloudFront distribution: a CDN distribution with the sample public bucket as an origin.
* Deployments bucket: The deployments bucket stores these CloudFormation templates, build artifacts, and other files used in infrastructure and deployment setup. Once this template is run, setup the below folder structure within it.
* Deployments bucket policy: A policy that prevents unencrypted uploads of content to certain folders within the bucket.

```
deployments bucket
|_cloudformation-templates
| |_infrastructure
| | |_VPC.json
| | |_IAM.json
| | |_S3.json
| | |_RDS.json
| | |_ElastiCache.json
| |_projects
| | |_web-project.json
|_credentials
| |_web-project.json
|_artifacts
| |_projects
|  |_web-project
|   |_deployment.zip
|_bootstraps
| |_apache_setup.sh
| |_logging.sh
```

### IAM
The IAM template defines a number of security and access management resources for the AWS account. These include:

* AWS Config Role: (Optional, enabled by setting the "CreateConfigRole" parameter to true). An IAM role with the necessary permissions for AWS Config to operate and save its logs to the logging bucket (name provided as a parameter).
* AWS CodeDeploy Role: (Optional, enabled by setting the "CreateCodeDeployRole" parameter to true). An IAM role with the necessary permissions for AWS CodeDeploy. This gives AWS permissions to perform the necessary steps to add life cycle hooks to auto scaling as well as download project artifacts from the deployments bucket.
* AWS CloudTrail CloudWatch Delivery Role: (Optional, enabled by setting the "CreateCloudTrailRole" parameter to true). An IAM role with the necessary permissions for AWS CloudTrail to send its logs to CloudWatch. This must be setup before CloudTrail can be integrated into CloudWatch (recommended). It assumes the default log stream name.
* AWS Instance Role: A sample role that demonstrates how instance roles can be created in CloudFormation. It is recommended to put these in the project-specific templates instead of the IAM template.
* Sample Cross Account Role: A sample role that demonstrates how cross-account access can be established. It includes a sample policy that allows all actions as long as an MFA token was used to login.
* Sample User: A sample traditional user account. These should only be used in cases where long-running AWS access keys and secrets are needed. They should never be used for hard-coding credentials in code. The user's access key and secret are printed as part of the template output.
* Jenkins User: This is a user account created for Jenkins with the typical set of permissions needed to launch CodeDeploy deployments, upload artifacts to S3 (the specific deployments bucket and artifacts folder), as well as upload new Lambda code (for automated Lambda deployments).
* Service accounts group: This is a sample group where non-human user accounts can be added for categorization.

### Route53
The Route53 template defines the root domain for the service as well as a sample subdomain. Generally, project specific subdomains should live in the project's template, but non-specific domains and subdomains should be defined here. This template also introduces another condition "IsDefault" which is used when the template is launched in a certain region (us-east-1 in this case). This is done because Route53 is a global service and the same root domain can not be created in multiple regions. This template defines:

* Main Hosted Zone: The main domain for your organization. This would be "example.com", "widgets.co", etc.
* Sample Blog Record: A sample record for a resource hosted at another organization (Squarespace in this case).

## RDS

Coming soon (relies on VPC)

## Project Templates

Coming soon
