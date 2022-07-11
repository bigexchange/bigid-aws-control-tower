# Automate BigID IAM role deployment in AWS accounts under AWS Control Tower

AWS Control Tower Lifecyle Integration with BigID - Allow new or updated AWS accounts in an AWS Control Tower to be automatically deployed with IAM roles. This role can be:

- **attached to a account-local BigID scanner** to read the data from various AWS resources.
- **assumed by centralized scanner** if you define a Scanner Account in stack parameters.

## Account-local scanners vs Cross account scanners
This template allows to either remote scanner in individual accounts or leverage cross-account authentication for centralized scanners.
Centralized scanning is preferable as it reduces the number of scanners to deploy and maintain. It is also mandatory if you plan to use BigID autodiscovery app via AWS Config aggregators.

* If you deploy an account-local scanner, use the BigID role name (as declared) in template parameters and attach it to your remote scanner. The role name must be available in all accounts
* If you deploy an cross-account scanner, you need to provide a specific accountID as the template parameter and deploy it there.

## Cross account scanning example
![](./assets/centralized%20scanning%20with%20AWS%20%2B%20BigID.png)

## How it Works

### Template internal machinery

BigID-Role-Deployment-template.yml template provisions infrastructure in the Control Tower Management account that allows automatic creation of BigID roles in control tower enrolled accounts.

* Creates a BigID Stackset in the Control Tower Management Account. This will deploy roles in enrolled accounts which can be further assumed by BigID scanner to read the data.
* Provisions a Amazon Eventbridge Rule that is triggered based on a Control Tower Lifecycle Event
* Provisions a Lambda as a target for the Eventbridge Events Rule. Each time an account is enrolled (created or updated) in control tower, the lambda create a new stack instance for this account.



## How to Install

### Prerequisites
1. Access to AWS Management account with Control Tower installed. The user should have write access to at least these CloudFormation, Lambda, Amazon Eventbridge & IAM services.
2. (Optional) Create (preferable) or identity a specific account for cross account scanning activities.

### Parameters
* BigIDRoleNme: IAM role name that will get created in all accounts except scanner account. Theses roles can be either directly attached to a scanner or assumed by scanner within the centralized scanning account.
    Default: "BigID"
* BigIDScannerRoleName: IAM role name for scanner that will get created only in scanner account
    Default: "BigIDScanner"
* BigIDScannerAccountID: Account number where the cross-account scanners will be deployed.

### Steps
1. Download the CloudFormation template BigID-Role-Deployment-template.yml
2. Launch the BigID-Role-Deployment-template.yml template in AWS Management account where Control Tower is deployed (same AWS region). Enter the optional IAM role name to be created.
3. Work with the BigID Services team to install BigID application and BigID scanners as applicable in your environment (based on your corporate policies, latency needs and sizing estimations). Whenever you want 

### How to use the roles
This integration will create 2 kind of roles, depending on accountID:
* an account-local scanner role that is deployed in all accounts except the dedicated scanner. This gives access to resources like S3, DynamoDB, etc...
* if you decide to use cross account scanning, a cross-account role is deployed in the Scanner account that will allow the scanner role to assume-role to all others Control Tower managed accounts.


**Test**

Test by creating a Lifecycle Event and add a managed account:

1. From the AWS Control Tower Management Account:
    - Use Account Factory or Service Catalog Account factory product to create a new managed account in the AWS Control Tower OR
    - Use Service Catalog (AccountFactory Product) to update an existing managed account - for e.g. change the OU of an existing managed account
 	- This can take up to ~20-30 minutes for the account to be successfully created and the AWS Control Tower Lifecycle Event to trigger
 	- Login to the AWS Control Tower managed account -
 		- Validate that an AWS CloudFormation stack instance has been provisioned that launches the BigID IAM role template in the managed account.
2. Create an AWS account and configure AWS Data-Sources:
    - Create an S3 bucket with some files (preferably containing sensitive info like credit-cards, email and/or DOB)
    - Create a DynamoDB table with data (preferably containing sensitive info like credit-cards, email and/or DOB)
3. Create an AWS account dedicated to BigID cross account scanning.
4. BigID Deployments:
    1. Account-local scanner
        - Within the data source account, deploy a remote scanner and attach the instance profile (name is defined by stack parameter RoleName, defaults to "BigID"). Use a specific scanner_group (example data_account)
        - Declare data sources (S3, DynamoDB) in BigID, with the same scanner group an use IAM Role as authentication method.
    2. Cross-account scanner
        - Within the BigID dedicated scanner account, deploy a remote scanner and attach the instance profile (name is define by stack parameter ScannerRoleName, defaults to "BigIDScanner"). Use a specific scanner_group (example centralized_scanner). For AWS Authentication Type:
        - Declare data sources (S3, DynamoDB) in BigID, with the same scanner group.
            - use "Cross Account Authentication"
            - Leave role session name blank
            - set AWS Role ARN to arn:aws:iam::*\<AccountID>*:role/*\<RoleName>* where AccountID is the Data source accountID and RoleName is the RoleName parameter value of the cloudformation stack you just deployed