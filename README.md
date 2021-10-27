
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

1.	Launch the **BigID-Role-Deployment-template.yml** template in AWS Management account where Control Tower is deployed(same aws region). Enter the optional IAM role name to be created.

2. <TODO - Big ID scanner deployment>


**Test** 

Test by creating a Lifecycle Event and add a managed account:

1. From the AWS Control Tower Management Account:
    - Use Account Factory or Service Catalog Account factory product to create a new managed account in the AWS Control Tower OR
    - Use Service Catalog (AccountFactory Product) to update an existing managed account - for e.g. change the OU of an existing managed account
 	- This can take up to ~20-30 minutes for the account to be successfully created and the AWS Control Tower Lifecycle Event to trigger
 	- Login to the AWS Control Tower managed account - 
 		- Validate that an AWS CloudFormation stack instance has been provisioned that launches the BigID IMA role template in the managed account. 
 		- < TODO - Big ID data collection test > Follow the step by step instructions in the BigID documentation 