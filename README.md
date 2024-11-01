# Guidance for Secure High-Performance Computing with NIST SP 800-223 on AWS
This solution aims to instruct and guide users how to build a cloud HPC for the new NIST SP 800-223 standard using AWS Parallel Cluster.
The National Institute of Standards and Technology (NIST) has published NIST SP 800-223: High-Performance Computing (HPC) Security: Architecture, Threat Analysis, and Security Posture. This new standard provides guidance on how to configure and secure a HPC cluster. This builder project aims to instruct and guide users on how to build a cloud HPC for the new NIST SP 800-223 compliance using AWS CloudFormation and AWS Parallel Cluster.

## Table of Contents (required)

List the top-level sections of the README template, along with a hyperlink to the specific section.

### Required

1. [Overview](#overview-required)
    - [Architecure](#architecture)
    - [Cost](#cost)
    - [AWS Services in this guidance](#aws-services-in-this-guidance)
3. [Prerequisites](#prerequisites-required)
    - [Operating System](#operating-system-required)
4. [Deployment Steps](#deployment-steps-required)
5. [Deployment Validation](#deployment-validation-required)
6. [Running the Guidance](#running-the-guidance-required)
7. [Next Steps](#next-steps-required)
8. [Cleanup](#cleanup-required)

***Optional***

8. [FAQ, known issues, additional considerations, and limitations](#faq-known-issues-additional-considerations-and-limitations-optional)
9. [Revisions](#revisions-optional)
10. [Notices](#notices-optional)
11. [Authors](#authors-optional)

## Overview (required)

1. Provide a brief overview explaining the what, why, or how of your Guidance. You can answer any one of the following to help you write this:

    - **Why did you build this Guidance?**
    - **What problem does this Guidance solve?**

2. Include the architecture diagram image, as well as the steps explaining the high-level overview and flow of the architecture. 
    - To add a screenshot, create an ‘assets/images’ folder in your repository and upload your screenshot to it. Then, using the relative file path, add it to your README. 

## TO DO UPDATE FOR THIS SPECIFIC GUIDANCE
## AWS Services in this Guidance

The following AWS Services are deployed in this Guidance:

| **AWS service**  | Description |
|-----------|------------|
|[Amazon VPC](https://aws.amazon.com/vpc/)|Core Service - provides additional Networking isolation and security | 
|[Amazon EC2](https://aws.amazon.com/ec2/)|Core Service - EC2 instances used as cluster nodes |
|[Amazon API Gateway](https://aws.amazon.com/api-gateway/)|Core service - create, publish, maintain, monitor, and secure APIs at scale|
|[Amazon Cognito](https://aws.amazon.com/cognito/)|Core service - provides user identity and access management (IAM) services|
|[Amazon Lambda](https://aws.amazon.com/lambda/)|Core service - provides serverless automation of user authentication|
|[Amazon FSx for Lustre](https://aws.amazon.com/fsx/lustre/)|Core service - provides high-performance Lustre file system |
|[Amazon Parallel Cluster](https://aws.amazon.com/hpc/parallelcluster/)|Core service - Open source cluster management tool for deployment and management of High Performance Computing (HPC) clusters |
|[Amazon High Performance Computing HPC cluster](https://aws.amazon.com/hpc/)|Core service - high performance compute resource|
|[Amazon System Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)|Auxiliary service - instance connection management|


## Plan your deployment

### Supported AWS Regions

This Guidance uses EC2 services with [specific instances such as `hpc6`](https://aws.amazon.com/ec2/instance-types/hpc6/) and FSx for Lustre storage services, which may not currently be available in all AWS Regions. You must launch this solution in an AWS Region where EC2 specific instance types and FSx is available. For the most current availability of AWS services by Region, refer to the [AWS
Regional Services
List](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/).

**Guidance for Building a High-Performance Numerical Weather Prediction System on AWS** is currently supported in the following AWS Regions (based on availability of [hpc6a](https://aws.amazon.com/ec2/instance-types/hpc6a/), [hpc7a](https://aws.amazon.com/ec2/instance-types/hpc7a/) and [hpc7g](https://aws.amazon.com/ec2/instance-types/hpc7g/) instances:


| AWS Region     | Amazon EC2 HPC Optimized Instance type                                                                                   |
| -------------- | ------------------------------------------------------------------------------------------------------------------------ |
| ap-southeast-1 | hpc6a.48xlarge                                                                                                           |
| eu-north-1     | hpc6id.32xlarge<br>  hpc6a.48xlarge                                                                                      |
| eu-west-1      | hpc7a.12xlarge<br> hpc7a.24xlarge<br> hpc7a.48xlarge<br> hpc7a.96xlarge<br>                                              |
| us-east-1      | hpc7g.4xlarge<br> hpc7g.8xlarge<br> hpc7g.16xlarge<br>                                                                   |
| us-east-2      | hpc6a.48xlarge <br> hpc6id.32xlarge <br> hpc7a.12xlarge <br> hpc7a.24xlarge <br> hpc7a.48xlarge <br> hpc7a.96xlarge <br> |

### Cost 

You are responsible for the cost of the AWS services used while running this guidance. 

Please refer to the [sample pricing webpage](https://calculator.aws/#/estimate?id=67ae5451a69c58dacd7dd7235a61834ae07e6f61) for each AWS Service used in this Guidance. Please note that monthly costs assume that an HPC cluster with Head Node of instanceType `c6a.2xlarge` and two Compute Nodes  of instanceType `hpc6a.48xlarge` with `1200 GB` of FSx for Lustre persistent storage provisioned for that cluster that are active 50%. In reality, computeNodes get de-provisonied around 10 min after completing a job and therefore monthly cost would be lower than this  estimate.

#### Sample cost table (Update needed) 

The following table provides a sample cost breakdown for an HPC cluster with one `c6a.2xlarge` Head Node and two Compute Nodes  of instanceType `hpc6a.48xlarge` with `1200 GB` of FSx for Lustre persistent storage allocated for it deployed in the `us-east-2` region: 


| Node Processor Type | On Demand Cost/month USD| 
| --------------------| ----------------- |
| *c6a.2xlarge*       |  $226.58            |   
| *hpc6a.48xlarge*    |  $2,102.40            |  
| *FSx Lustre storage*|  $720.07           |  
| *VPC, subnets*|  $283.50           |  
| *Total estimate*|  $3,332.55       |

### Security

When you build systems on AWS infrastructure, security responsibilities are shared between you and AWS. This [shared responsibility
model](https://aws.amazon.com/compliance/shared-responsibility-model/) reduces your operational burden because AWS operates, manages, and
controls the components including the host operating system, the virtualization layer, and the physical security of the facilities in
which the services operate. For more information about AWS security, visit [AWS Cloud Security](http://aws.amazon.com/security/).

[AWS ParallelCluster](https://aws.amazon.com/hpc/parallelcluster/) users are securely authenticiated and authorized to their roles via [Amazon Cognito](https://aws.amazon.com/cognito/) user pool service. HPC cluster EC2 components are deployed into a Virtual Private Cloud (VPC) which provides additional network security isolation for all contained components. Head Node is depoyed into a Public subnet and available for access via secure connections (SSH and DCV), compute nodes are deployed into Private subnet and managed from Head node via SLURM package manager. Data stored in Amazon FSx for Lustre is [enrypted at rest and in transit](https://docs.aws.amazon.com/fsx/latest/LustreGuide/encryption-fsxl.html).

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## Deployment Steps
<!-- TO DO: update the link once is published -->

<b>Please see published [Implementation Guide](https://aws-solutions-library-samples.github.io/compute/building-a-high-performance-numerical-weather-prediction-system-on-aws.html#create-and-connect-to-the-hpc-cluster) for step-by-step deployment instructions for this guidance.</b>



## END TO DO

## Deployment Steps (required)

Deployment steps must be numbered, comprehensive, and usable to customers at any level of AWS expertise. The steps must include the precise commands to run, and describe the action it performs.

* All steps must be numbered.
* If the step requires manual actions from the AWS console, include a screenshot if possible.
* The steps must start with the following command to clone the repo. ```git clone xxxxxxx```
* If applicable, provide instructions to create the Python virtual environment, and installing the packages using ```requirement.txt```.
* If applicable, provide instructions to capture the deployed resource ARN or ID using the CLI command (recommended), or console action.

 
**Example:**

1. Clone the repo using command ```git clone xxxxxxxxxx```
2. cd to the repo folder ```cd <repo-name>```
3. Install packages in requirements using command ```pip install requirement.txt```
4. Edit content of **file-name** and replace **s3-bucket** with the bucket name in your account.
5. Run this command to deploy the stack ```cdk deploy``` 
6. Capture the domain name created by running this CLI command ```aws apigateway ............```



## Deployment Validation  (required)

<Provide steps to validate a successful deployment, such as terminal output, verifying that the resource is created, status of the CloudFormation template, etc.>


**Examples:**

* Open CloudFormation console and verify the status of the template with the name starting with xxxxxx.
* If deployment is successful, you should see an active database instance with the name starting with <xxxxx> in        the RDS console.
*  Run the following CLI command to validate the deployment: ```aws cloudformation describe xxxxxxxxxxxxx```



## Running the Guidance (required)

<Provide instructions to run the Guidance with the sample data or input provided, and interpret the output received.> 

This section should include:

* Guidance inputs
* Commands to run
* Expected output (provide screenshot if possible)
* Output description



## Next Steps (required)

Provide suggestions and recommendations about how customers can modify the parameters and the components of the Guidance to further enhance it according to their requirements.


## Cleanup (required)

- Include detailed instructions, commands, and console actions to delete the deployed Guidance.
- If the Guidance requires manual deletion of resources, such as the content of an S3 bucket, please specify.



## FAQ, known issues, additional considerations, and limitations (optional)


**Known issues (optional)**

<If there are common known issues, or errors that can occur during the Guidance deployment, describe the issue and resolution steps here>


**Additional considerations (if applicable)**

<Include considerations the customer must know while using the Guidance, such as anti-patterns, or billing considerations.>

**Examples:**

- “This Guidance creates a public AWS bucket required for the use-case.”
- “This Guidance created an Amazon SageMaker notebook that is billed per hour irrespective of usage.”
- “This Guidance creates unauthenticated public API endpoints.”


Provide a link to the *GitHub issues page* for users to provide feedback.


**Example:** *“For any feedback, questions, or suggestions, please use the issues tab under this repo.”*

## Revisions (optional)

Document all notable changes to this project.

Consider formatting this section based on Keep a Changelog, and adhering to Semantic Versioning.

## Notices (optional)

Include a legal disclaimer

**Example:**
*Customers are responsible for making their own independent assessment of the information in this Guidance. This Guidance: (a) is for informational purposes only, (b) represents AWS current product offerings and practices, which are subject to change without notice, and (c) does not create any commitments or assurances from AWS and its affiliates, suppliers or licensors. AWS products or services are provided “as is” without warranties, representations, or conditions of any kind, whether express or implied. AWS responsibilities and liabilities to its customers are controlled by AWS agreements, and this Guidance is not part of, nor does it modify, any agreement between AWS and its customers.*


## Authors (optional)

Name of code contributors
## License

This library is licensed under the MIT-0 License. See the LICENSE file.
