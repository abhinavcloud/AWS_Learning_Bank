# AWS Organizations

AWS Organizations helps centrally manage and govern your environment as organizations grow and scale AWS resources. Using Organizations, companies can create accounts and allocate resources, group accounts to organize workflows, apply policies for governance, and simplify billing by using a single payment method for all accounts.

## AWS Organizations can:
- Add accounts
- Group accounts
- Apply policies
- Enable AWS services

## Features of AWS Organizations:
- Manage AWS Accounts for permissions, security, costs and workloads.
- Create new accounts, group them into organizational units (OUs). 
- Apply tag policies to enforce tags across all resources in each AWS account across all OUs.
- Delegate responsibility for supported AWS services to accounts so that user can manage them on organization behalf.
- Secure and monitor accounts centrally and mitigate threats with Amazon GuardDuty, IAM Access Analyzer for restricting unintended access and Amazon Macie to remove PII data/.
- Set up AWS IAM Identity Center to provide access to AWS resources in accounts and apply organizational policies and Service Control Policies (SCPs) to control access to AWS resources, Resource Control Policies (RCPs) to centrally prevent unintended use of AWS resources.
- Audit accounts for compliance using AWS Cloudtrail centrally which cannot be turned off by members accounts.
- Centrally manage billing and costs using AWS Cost Explorer and optimize compute resources using AWS Compute Optimizer

## Organization Structure

                                                Organization
                                                    |
    ________________________________________________|______________________________________________________________
    |                            |                                           |                                     |
    Account A          Management Account (Root)                          Organizational Unit 1                 Organizational Unit 2
                        - Service Control Policies                           |                                     |
                        - Resource Policies                 _________________|_________________         ___________|_____________
                        - Declarative Policies              |                |                 |        |           |            |
                                                        Account B          OU 3               OU 4     OU 5        OU 6          OU 7
                                                                             |
                                                                    _________|_________
                                                                    |                  |
                                                                OU 8                  OU 9   


## Service Control Policies
Service Control Policies are types of organizational policies that we can use to manage permissions in organizations. SCP offer central control over the maximum available permissions for the IAM users and IAM roles in organizations.

For a permission to be allowed for a specific account, there must be an explicit Allow statement at every level from the root through each OU in the direct path to the account (including the target account itself). This is why when you enable SCPs, AWS Organizations attaches an AWS managed SCP policy named FullAWSAccess which allows all services and actions. If this policy is removed and not replaced at any level of the organization, all OUs and accounts under that level would be blocked from taking any actions.


SCP evaluation follows a deny-by-default model, meaning that any permissions not explicitly allowed in the SCPs are denied. If an allow statement is not present in the SCPs at any of the levels such as Root, Production OU 1 or Account B, the access is denied.

For a permission to be denied for a specific account, any SCP from the root through each OU in the direct path to the account (including the target account itself) can deny that permission.

For example, let’s say there is an SCP attached to the Production OU that has an explicit Deny statement specified for a given service. There also happens to be another SCP attached to Root and to Account B that explicitly allows access to that same service, as shown in Figure 3. As a result, both Account A and Account B will be denied access to the service as a deny policy attached to any level in the organization is evaluated for all the OUs and member accounts underneath it.


Effecitve Strategy is to use explicit denies at OU levels to restrict the permissions to the accounts under the OU. 

Alternativeely,  AWSFullAccess can be replaced by specific Allow Policies which allow specific actions on sepecific resources on accounts under the policies. In this case, the same Allow lists should be applied at each level so that that allows trickles down successfully to each account. Even if FullAWSAccess is attached at root, if any subsequent OU doesnt have the FullAWSAccess and ony specific Allow lists, then each account should also have those Allow lists for those specific resources and actions.

[Please refer to the following links for eaxmple on SCP Evaluations](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_evaluation.html)


## Resource Control Policies
The RCPFullAWSAccess policy is an AWS managed policy. It is automatically attached to the organization root, every OU, and every account in your organization, when you enable resource control policies (RCPs). You cannot detach this policy. This default RCP allows all principals and actions access to pass through RCP evaluation, meaning until you start creating and attaching RCPs, all your existing IAM permissions continue to operate as they did. This AWS managed policy does not grant access.

You can make use of Deny statements to block access to resources in your organization. For a permission to be denied for a resource in a specific account, any RCP from the root through each OU in the direct path to the account (including the target account itself) can deny that permission.

Deny statements are a powerful way to implement restrictions that should be true for a broader part of your organization. For example, you can attach a policy to help prevent identities external to your organization from accessing your resources root level, and that will be effective for all accounts in the organization. 


