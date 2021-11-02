
# Automate BigID IAM role deployment in AWS accounts under AWS Control Tower

* AWS Control Tower Lifecyle Integration with BigID - Allow new or updated AWS accounts in an AWS Control Tower to be automatically deployed with IAM role. This role will be assumed by BigID scanners to read the data from various AWS resources.

## How it Works

1. **Template: BigID-Role-Deployment-template.yml**:
 * This template provisions infrastructure in the Control Tower Management account that allows creation of BigID scanner role in Control Tower managed accounts whenever a new Control Tower managed account is added & existing account is updated
 * Creates a BigID Stackset in the Control Tower Management Account 
 * Provisions a Amazon Eventbridge Rule that is triggered based on a Control Tower Lifecycle Event
 * Provisions a Lifecyle Lambda as a target for the Eventbridge Events Rule
 	- The Lifecycle Lambda deploys a BigID stack in the newly added Control Tower managed account--thus placing that account under BigID monitoring
 * The infrastructure provisioned by this template above allows for a Control Tower lifecycle event trigger specifically the CreateManagedAccount or UpdateManagedAccount events to:
	- Trigger the Lifecyle Lambda that creates BigID stack instance in the managed account based on the BigID stackset in the management account
 * This stack instance in new managed account created IAM role which can be further assumed by BigID scanner to read the data.


## How to Install


**Prerequisites**
1. Access to AWS Management account with Control Tower installed. The user should have write access to at least these CloudFormation, Lambda, Amazon Eventbridge & IAM services.

### Steps: 
1. Download the CloudFormation template BigID-Role-Deployment-template.yml
2. The default IAM policy BigIDMonitoringPolicy in this template allow open read-only access to all the AWS services supported by BigID. If you want to narrow down this access to certain services, please edit the section 'BigIDMonitoringPolicy' from above template. For more information on IAM policies click [here](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_examples.html)
3. Launch the BigID-Role-Deployment-template.yml template in AWS Management account where Control Tower is deployed (same AWS region). Enter the optional IAM role name to be created.
4. Work with the BigID Services team to install BigID application and BigID scanners as applicable in your environment (based on your corporate policies, latency needs and sizing estimations). All BigID components are deployed as container images and managed either via Docker-Compose on AWS EC2 OR using a Kubernetes on AWS EKS.
5. Configure a BigID Scanner (on EC2 or EKS) and ensure they are mapped to the AWS IAM Role created using the template mentioned above (default BigIDScannerRole)
6. Always use the "IAM Role Authentication" method while creating any AWS data-sources.

Note: This is just a starter policy, please review it thoroughly with your System-Admin/DevOps team.


**Test** 

Test by creating a Lifecycle Event and add a managed account:

1. From the AWS Control Tower Management Account:
    - Use Account Factory or Service Catalog Account factory product to create a new managed account in the AWS Control Tower OR
    - Use Service Catalog (AccountFactory Product) to update an existing managed account - for e.g. change the OU of an existing managed account
 	- This can take up to ~20-30 minutes for the account to be successfully created and the AWS Control Tower Lifecycle Event to trigger
 	- Login to the AWS Control Tower managed account - 
 		- Validate that an AWS CloudFormation stack instance has been provisioned that launches the BigID IMA role template in the managed account. 
2. Create an AWS account and configure AWS Data-Sources:
    - Create an S3 bucket with some files (preferably containing sensitive info like credit-cards, email and/or DOB)
    - Create a DynamoDB table with data (preferably containing sensitive info like credit-cards, email and/or DOB)
3. BigID Configuration:
    - Download and install a BigID (set of docker images) on a EC2 instance
    - Download and install a BigID Scanner (docker image) on a separate EC2 instance
    - Map the BigID Scanner to the AWS IAM Role created using the template mentioned above (default BigIDScannerRole)
    - Add two new AWS data-sources on BigID (one for S3 and one for DynamoDB) and sse the "IAM Role Authentication" method for credentials. 
