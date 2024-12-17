# Guidance for Deploying Secure High-Performance Computing Clusters on AWS

This guidance aims to instruct and guide users how to build a cloud HPC for the new NIST SP 800-223 standard using [AWS Parallel Cluster](https://aws.amazon.com/hpc/parallelcluster/).
The [National Institute of Standards and Technology (NIST)](https://www.nist.gov/) has published [NIST SP 800-223: High-Performance Computing (HPC) Security: Architecture, Threat Analysis, and Security Posture](https://www.nist.gov/news-events/news/2024/02/high-performance-computing-security-nist-releases-sp-800-223) standard. This new standard provides guidance on how to configure and secure an HPC cluster. This guidance instructs users on how to build a cloud HPC for the new NIST SP 800-223 compliance using [AWS CloudFormation](https://aws.amazon.com/cloudformation/) and AWS Parallel Cluster.

## Table of Contents

<!-- List the top-level sections of the README template, along with a hyperlink to the specific section -->

### Table of Contents

- [Guidance for Secure High-Performance Computing with NIST SP 800-223 on AWS](#guidance-for-secure-high-performance-computing-with-nist-sp-800-223-on-aws)
  - [Table of Contents (required)](#table-of-contents-required)
    - [Required](#required)
  - [Overview](#overview)
  - [Architecture Overview](#architecture-overview)
    - [Architecture diagrams](#architecture-diagrams)
    - [Cost](#cost)
    - [Sample Cost Table](#cost-table)
    - [Security](#security)
  - [Prerequisites](#prerequisites)
    - [SSH Access](#ssh-access)
    - [AWS account requirements (If applicable)](#aws-account-requirements-if-applicable)
  - [Deployment Steps](#deployment-steps)
    - [Default Stack Names](#default-stack-names)
  - [Deployment Validation](#deployment-validation)
  - [Next Steps](#next-steps)
    - [Login via SSM](#login-via-ssm)
    - [Login via SSH](#login-via-ssh)
  - [Cleanup](#cleanup)
  - [Notices](#notices)
  - [Authors](#authors)
  - [License](#license)

## Overview

Amazon Web Services (AWS) provides the most elastic and scalable cloud infrastructure to run your hpc workloads. With virtually unlimited capacity - engineers, researchers, HPC system administrators, and organizations can innovate beyond the limitations of on-premises HPC infrastructure.

High Performance Compute (HPC) on AWS removes the long wait times and lost productivity often associated with on-premises HPC clusters. Flexible HPC cluster configurations and virtually unlimited scalability allows you to grow and shrink your infrastructure as your workloads dictate, not the other way around.

This guidance provides a comprehensive approach to deploying a secure, compliant, and high-performance HPC environment on AWS. It addresses the unique security challenges of HPC systems while maintaining the performance requirements critical for computationally intensive workloads.

We developed this guidance in response to the growing need for secure HPC environments in cloud settings. Many organizations, especially those in research, engineering, and data-intensive fields, require immense computational power but struggle to balance this with stringent security and compliance requirements. The NIST SP 800-223 publication provides an excellent framework for addressing these challenges, and we wanted to demonstrate how to implement these recommendations using AWS services.

## Architecture Overview

### Architecture diagrams

Architecture diagrams below show sample NIST 800-223 based infrastructure architecture, provisoning and deployment process using cloudformation, HPC Cluster deployment, and user interactions via AWS ParallelCluster.  Depending on the region you deploy the guidance in, it will automatically scale to from 2-4 AZs in order to maximize availability and redundancy of your cluster.

<!-- ![Reference Architecture](/assets/images/ref-arch.png "Reference Architecture") -->
![Reference Architecture](/assets/images/deploying-high-performance-computing-clusters-on-aws-hpc-cluster-deployment1.png "Reference Architecture p 1")

(1) Admin/DevOps users can deploy this architecture using a series of AWS CloudFormation templates. These templates provision networking resources, including [Amazon Virtual Private Cloud (Amazon VPC)](https://aws.amazon.com/cloudformation/) and subnets. The templates also provision resources for security and storage, such as [Amazon Simple Storage Service (Amazon S3)](https://aws.amazon.com/s3/), [Amazon Elastic File System (Amazon EFS)](https://aws.amazon.com/efs/), and [Amazon FSx for Lustre](https://aws.amazon.com/fsx/lustre/). There are optional templates included to deploy a Slurm accounting database (DB) and a Microsoft Active Directory user directory. <br/>
(2) Four logical subnets (zones) are created, each in multiple [Availability Zones (AZs)](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/?id=Availability_Zones), based on the target AWS Region. All required networking, networking access control list (ACLs), routes, and security resources are deployed. deployed subnets for the Storage Zone. An FSx for Lustre file system is created that is used as a highly performant scratch file system in the preferred AZ. The four zones are: 1) Access Zone (public subnet), 2) Compute Zone, 3) Management Zone, and 4) Storage Zone (all private subnets).<br/>
(3) An [Amazon RDS for MySQL](https://aws.amazon.com/rds/mysql/) instance is created that will be used as the Slurm Accounting Database. This is set up in a single zone, or can be modified to be multi-AZ if preferred. One [AWS Directory Service](https://aws.amazon.com/directoryservice/) user directory is created across two AZs.<br/>
(4) An Amazon EFS file system is created for shared cluster storage that is mounted in all of the deployed subnets for the Storage Zone. An FSx for Lustre file system is created that is used as a highly performant scratch file system in the preferred AZ.<br/>
(5) Two Amazon S3 buckets are created: one for campaign storage using [Amazon S3 Intelligent-Tiering](https://aws.amazon.com/s3/storage-classes/intelligent-tiering/), and one for archival storage using Amazon S3 Glacier.<br/>
(6) Random passwords are generated for both the Slurm accounting database and the Directory Service that are stored securely in [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/).<br/>

![Reference Architecture](/assets/images/deploying-high-performance-computing-clusters-on-aws-hpc-cluster-deployment2.png "Reference Architecture p 2")

(1) Admin/DevOps users use the [AWS ParallelCluster](https://aws.amazon.com/hpc/parallelcluster/) AWS CloudFormation stack to deploy HPC resources. Resources can reference the network, storage, security, database, and user directory from the previously launched CloudFormation stacks. <br/>
(2) The AWS ParallelCluster CloudFormation template provisions a sample cluster configuration, which includes a head node deployed in a single Availability Zone within the Management zone. It also provisions a login node deployed in a single Availability Zone within the Access zone. <br/>
(3) The Slurm workload manager is deployed on the head node and used for managing the HPC workflow processes. <br/>
(4) The sample cluster configuration included creates two Slurm queues that provision compute nodes within the Compute zone. One queue uses compute-optimized [Amazon Elastic Compute Cloud (Amazon EC2)](https://aws.amazon.com/ec2/) instances, while the other queue utilizes GPU-accelerated EC2 instances. <br/>
(5) Users access this guidance by establishing a connection to the deployed login node within the Access zone, utilizing either a [NICE DCV](https://www.ni-sp.com/products/nice-dcv/), SSH, or an [AWS Systems Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html). <br/>
(6) Users authenticate to the log in node using a username and password stored in the [AWS Managed Microsoft Active Directory](https://docs.aws.amazon.com/directoryservice/latest/admin-guide/directory_microsoft_ad.html). <br/>

### Cost

You are responsible for the cost of the AWS services used while running this Guidance. As of November 2024, the cost for running this Guidance with the default settings in the US East (N. Virginia) region is approximately $1,156 per month.

We recommend creating a [Budget](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html) through [AWS Cost Explorer](https://aws.amazon.com/aws-cost-management/aws-cost-explorer/) to help manage costs. Prices are subject to change. For full details, refer to the pricing webpage for each AWS service used in this Guidance.

### Cost Table

The following table provides a sample cost breakdown for deploying this guidance with the default parameters in the US East (N. Virginia) `us-east-1` Region for one month.

| Stack Name  | AWS Services | Cost [USD] |
| ----------- | ------------ | ------------ |
| Network | VPC, Subnets, NAT Gateway, VPC Endpoints  | $596.85/month |
| Security | Security Groups | $0.00/month |
| Storage | S3, EFS, FSx, EBS | $172.19/month |
| Slurm Accounting | RDS Database | $73.84/month |
| Active Directory | Managed AD (Enterprise)  | $288.00/month |
| Cluster | Head node, Login node | $25.00/month |
| **Total** |  | **$1155.88/month** |

***Note: This focus of this Guidance is to provide an example of securing the underlying AWS services and infrastructure that an HPC cluster will eventually run on.  It does not aim to include any costs related to running an actual HPC workload.  Please use the [AWS Pricing Calculator](https://calculator.aws/) to estimate any additional costs related to your specific HPC workload usecase.***

### Security

When you build systems on AWS infrastructure, security responsibilities are shared between you and AWS. This [shared responsibility
model](https://aws.amazon.com/compliance/shared-responsibility-model/) reduces your operational burden because AWS operates, manages, and
controls the components including the host operating system, the virtualization layer, and the physical security of the facilities in
which the services operate. For more information about AWS security, visit [AWS Cloud Security](http://aws.amazon.com/security/).

[AWS ParallelCluster](https://aws.amazon.com/hpc/parallelcluster/) users can be securely authenticiated and authorized using [Amazon Manageged Microsoft Active Directory](https://aws.amazon.com/directoryservice/). HPC cluster EC2 components are deployed into a Virtual Private Cloud (VPC) which provides additional network security isolation for all contained components. Login Node is depoyed into a Public subnet and available for access via secure connections (SSH and SSM), Head Node is depoyed into a Private subnet and available for access via secure connections (SSH and SSM), compute nodes are deployed into Private subnet and managed from Head node via SLURM package manager, Slurm accounting database is deployed into a Private subnet and managed from the Head node using Slurm. Data stored in Amazon S3, Amazon EFS, and Amazon FSx for Lustre is [enrypted at rest and in transit](https://docs.aws.amazon.com/whitepapers/latest/logical-separation/encrypting-data-at-rest-and--in-transit.html).  Access to other AWS services from AWS ParallelCluster components are secured over [VPC Endpoints](https://docs.aws.amazon.com/whitepapers/latest/aws-privatelink/what-are-vpc-endpoints.html) from a Private management subnet.

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## Prerequisites

### Secure Shell (SSH) Access

If you prefer to use SSH to access the login node or head node you will need to create a new SSH keypair in your account ***before*** launching the ParallelCluster CloudFormation template.

To do that, please follow the steps below:

1. Login to your AWS account
2. In the search bar at the top of the screen type in EC2
3. In the list of services select EC2
4. In the left-hand menu select Key Pairs under the Network & Security section
5. Click Create key pair
6. Enter a key pair name
7. Select your preferred key pair type and format and click Create key pair
8. This will automatically start a download of the private key for the key pair you just created
9. Save this key in a secure location (this key can act as your password to login to the nodes launched by this template)

### AWS account requirements

This deployment requires you have access to Amazon CloudFormation in your AWS account with permissions to create the following resources”

**AWS Services in this Guidance:**

- [Amazon VPC](https://aws.amazon.com/vpc/)
- [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/)
- [Amazon Identity and Access Management (IAM)](https://aws.amazon.com/iam/)
- [Amazon Elastic Compute Cloud (EC2)](https://aws.amazon.com/ec2/)
- [Amazon Elastic File System (EFS)](https://aws.amazon.com/efs/)
- [Amazon Elastic Block Store (EBS)](https://aws.amazon.com/ebs/)
- [Amazon FSx for Lustre (FSxL)](https://aws.amazon.com/fsx/lustre/)
- [Amazon Relational Database Service (RDS)](https://aws.amazon.com/rds/)
- [Amazon Dirctory Service](https://aws.amazon.com/directoryservice/)
- [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/)
- [AWS Systems Manager](https://aws.amazon.com/systems-manager/)
- [All service used by AWS ParallelCluster](https://aws.amazon.com/hpc/parallelcluster/)

## Deployment Steps

1. Clone the sample  code repository:
   - ```git clone https://github.com/aws-solutions-library-samples/guidance-for-building-nist-sp-800-223-hpc-on-aws.git```
2. Change Directory to the deployment folder inside the repo
   - ```cd guidance-for-building-nist-sp-800-223-hpc-on-aws/deployment```
3. Locate the six Amazon CloudFormation templates and review them in order in a text editor of your choice or in the Amazon CloudFormation console
   - In most cases you will want to use the default settings, however, you have the ability to modify these templates to your specific needs.
Below are default stack source file names with corresponding CFN stack names:

### Default Stack Names

| Template File Name  | Stack Name  |
| ----------- | ------------ |
| 0_network.yaml | nist-network  |
| 1_security.yaml | nist-security  |
| 2_storage.yaml | nist-storage  |
| 3_slurm_db.yaml | nist-database  |
| 4_active_directory.yaml | nist-ad  |
| 5_pcluster.yaml | nist-hpc  |

4. Open a browser and login to your AWS Account
5. Locate the search bar at the top of your screen and type in `CloudFormation`
6. When presented with a list of services click CloudFormation to open the CloudFormation console

![CloudFormation](/assets/images/deployment_steps/0_deployment.png)

7. Click the `Create Stack` button

![Create Stack](/assets/images/deployment_steps/1_deployment.png)

8. In the `Prepare template` section select `Choose an existing template`:

![Prepare Tempalte](/assets/images/deployment_steps/2_deployment.png)

9. In the `Specify template` section select `Upload a template file`
10. Click the `Choose file` button

![Choose File](/assets/images/deployment_steps/3_deployment.png)

11. Navigate to the location on your local computer where you cloned the sample code and navigate into the `deployment` folder.
    There you will find the CloudFormation templates prefaced with a number that will indicate the order to execute them in.
13. Select the first template titled `0_network.yaml`
14. For each template you will be asked to provide a Stack name, this name must be a unique stack name for the region you are deploying in.

***Important: The stack name should be noted for use in deployment of other templates. Downstream services will need to know this stack name in order to reference Amazon Resource Names (ARNs) or resource IDs that will be exported/output for each template***

![Stack Name](/assets/images/deployment_steps/4_deployment.png)

15. For the `Network` stack, review the parameters and adjust as needed based on your specific use case or requirements:  
16. Once you have reviewed and validated the parameters, click the `Next` button at the bottom of the page
17. Leave the default options on the `Configure stack options` page
18. You will need to scroll to the bottom of this page and select the check box to allow CloudFormation to create IAM resources on your behalf
19. Click `Next`

![Choose File](/assets/images/deployment_steps/5_deployment.png)

19. On the `Review and create` screen review your selections again and then click the Submit button at the bottom of the page:

![Submit](/assets/images/deployment_steps/6_deployment.png)

20. Your CloudFormation stack will begin deploying
21. You can monitor the deployment progress in the AWS CloudFormation console

![Choose File](/assets/images/deployment_steps/7_deployment.png)

22. Wait until you see the stack status update from "CREATE_IN_PROGRESS" to "CREATE_COMPLETE" before moving on to deploying the next template
23. You can review the outputs generated by the stack by going to the `Outputs` tab for each stack or going to the `Exports` page on the left-hand menu
    - ***Note: The export values will be used by later templates to reference resources created in earlier templates***

Outputs View:

![Outputs](/assets/images/deployment_steps/8_deployment.png)

Exports View:
   
![Exports](/assets/images/deployment_steps/9_deployment.png)

24.  Repeat the steps above starting with step 7. moving on to the next stack in the `deployment` folder (listed in Step 3. above)

***Important: Stacks 1-5  have input parameters that asks for the previous stack names.  If you modify the stack names from their default values, you will need to also update the parameters in each subsequent stack with the appropriate name so that the relevant services can be referenced.***

Below as in example of input parameters for the `hpc-pc-cluster` stack: 

<img src="/assets/images/deployment_steps/nist-hpc-pccluster-parameters.jpg" alt="HPC cluster input parameters" width="650" height="400">

***Note: The storage, Slurm database, Active Directory, and AWS ParallelCluster stacks are intended to be simple examples for testing the NIST SP 800-223 reference architecture.  For more production ready versions of these templates see AWS [HPC Recipes](https://github.com/aws-samples/aws-hpc-recipes/tree/main/recipes) repository***


## Deployment Validation 

* Open CloudFormation console and verify the status of the guidance templates with the names listed in Step 3.

<!-- img src="/assets/images/deployment_steps/0_validate.png" alt="Validate" width="200" height="325" -->

<img src="/assets/images/deployment_steps/nist-hpc-stacks_completed.jpg" alt="Validate deployment" width="250" height="525">

* Make sure that all CloudFormation stacks are deployed successfully, with resulting status of "CREATE_COMPLETE"

You can also open a nested stack whose name starts with `c-nist` and look into its `Outputs` tab to get values of some important cluster parameters:

<img src="/assets/images/deployment_steps/c-nist-hpc-cluster-deployed.jpg" alt="Nested Stack Cluster parameters" width="830" height="525">

If you need to verify that specified high performance FSx storage was provisioned, navigate to the [FSx section of AWS Console](https://us-east-1.console.aws.amazon.com/fsx/home?region=us-east-1#file-systems) and look for the FSx for Lustre entry like:

<img src="/assets/images/deployment_steps/nist-hpc-storage-fsxl.jpg" alt="FSxL Storage" width="760" height="525">

## Next Steps 

Now that you have successfully deployed the infrastructure need to comply with the guidelines and recommendations outlined in the [NIST SP 800-223](https://www.nist.gov/news-events/news/2024/02/high-performance-computing-security-nist-releases-sp-800-223).

You can begin using the cluster by logging into either its Login node to submit a job or to the management node to review or modify any of the Slurm settings.  You can use SSM to securely open a terminal session to either the Login node or the Head node by:

<img src="/assets/images/deployment_steps/nist-head-login-nodes.jpg" alt="HPC Login and Head Nodes">

### Login via AWS Systems Manager (SSM) 

1. In the search bar above type in `EC2`
2. In the list of services select `EC2`
3. On the left hand menu select `Instances`
4. Locate either the head node or the Login node and select that instance by checking the box to the left of it
5. Locate the `Connect` button near the top of the screen
6. In the window that opens click the `Session Manager` tab
7. Click the `Connect` button to open a secure terminal session in your browser

<img src="/assets/images/deployment_steps/nist-hpc-login_node_connect.jpg" alt="SSM Connect Head Node"> 

### Login via SSH

Alternatively, when you launch the `5_pcluster.yaml` CloudFormation template, you can select an SSH Key pair that already exists in your AWS Account.  If you completed the prerequistes steps to create a key pair you will see it populated in this list.

1. Locate your ssh key pair
2. Ensure you have the proper permissions set on the key pair (read-only access)
   ```chmod 400 /path/key/ssh_key.pem```
3. In the list of services select `EC2`
4. On the left hand menu select `Instances`
5. Locate either the head node or the login node and select one instance by checking the box to the left of the instance
6. Locate the `Connect` button near the top of the screen
7. In the window that opens click the SSH client tab
8. Follow the instructions on the screen to login to your instance

### Run Test SLURM Job
You can follow steps in [AWS documentation](https://docs.aws.amazon.com/parallelcluster/latest/ug/tutorials-running-your-first-job-on-version-3.html#tutorial-1stjob-first-slurm-job) to run a simple job using Slurm workload scheduler.
You need to be using `ec2-user` user to get access to all pre-installed HPC utilities, that can be achieved by changing shell prompt:

```
sudo su ec2-user
```
After that, you can run commands below to check HPC Cluster capacity, compute module availability, create a test `Hello World` job:

```
sinfo
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
cpu*         up   infinite      2  idle~ cpu-dy-cpu-[1-2]
gpu          up   infinite      2  idle~ gpu-dy-gpu-[1-2]
[ec2-user@ip-10-0-2-161 ~]$ !3
module avail

------------------------------------------------------------------------ /opt/amazon/modules/modulefiles -------------------------------------------------------------------------
libfabric-aws/1.22.0amzn1.0 openmpi/4.1.6               openmpi5/5.0.3

------------------------------------------------------------------------- /usr/share/Modules/modulefiles -------------------------------------------------------------------------
dot         module-git  module-info modules     null        use.own

-------------------------------------------------------------------- /opt/intel/mpi/2021.13/etc/modulefiles/ ---------------------------------------------------------------------
intelmpi/2021.13
[ec2-user@ip-10-0-2-161 ~]$ cat hellojob.sh
#!/bin/bash
sleep 30
echo "Hello World from $(hostname)"
[ec2-user@ip-10-0-2-161 ~]$ sbatch ./hellojob.sh
Submitted batch job 1
[ec2-user@ip-10-0-2-161 ~]$ squeue
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
                 1       cpu hellojob ec2-user CF       0:08      1 cpu-dy-cpu-1
[ec2-user@ip-10-0-2-161 ~]$ squeue
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
                 1       cpu hellojob ec2-user CF       0:26      1 cpu-dy-cpu-1
[ec2-user@ip-10-0-2-161 ~]$ squeue
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
                 1       cpu hellojob ec2-user CF       0:28      1 cpu-dy-cpu-1
[ec2-user@ip-10-0-2-161 ~]$ squeue
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
                 1       cpu hellojob ec2-user CF       0:33      1 cpu-dy-cpu-1
```
Wait until the job completes (no output of `squeue` command) and then check job output:
```
squeue
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
ls -l
total 8
-rwxrwxr-x 1 ec2-user ec2-user 57 Dec 16 23:43 hellojob.sh
-rw-rw-r-- 1 ec2-user ec2-user 32 Dec 16 23:47 slurm-1.out
[ec2-user@ip-10-0-2-161 ~]$ cat slurm-1.out
Hello World from ip-10-0-54-243
```
This confirms that your provisioned HPC cluster can run a compute job on its node(s).

## Cleanup

1. In the AWS Management Console, navigate to CloudFormation and locate the 6 guidance stacks deployed.
2. Starting with the most recent stack (not including any nested stacks), select the stack and click `Delete`
3. Repeat this for each of the 6 CloudFormation stacks deployed to remove all resources from your account
   
## Notices

Customers are responsible for making their own independent assessment of the information in this Guidance. This Guidance: (a) is for informational purposes only, (b) represents AWS current product offerings and practices, which are subject to change without notice, and (c) does not create any commitments or assurances from AWS and its affiliates, suppliers or licensors. AWS products or services are provided “as is” without warranties, representations, or conditions of any kind, whether express or implied. AWS responsibilities and liabilities to its customers are controlled by AWS agreements, and this Guidance is not part of, nor does it modify, any agreement between AWS and its customers.

## Authors

Chris Riddle - Sr. Solutions Architect AWS <br/>
Daniel Zilberman - Sr. Solutions Architect AWS

## License

This library is licensed under the MIT-0 License. See the [LICENSE](LICENSE) file.
