---
layout: post
title: AWS Identity and Access Management
categories: [ AWS ]
tags: [ AWS, IAM, Access Management, Identify, IAM Roles ]
enable_toc: true
render_with_liquid: true
date: 2023-04-06 00:00:00 +0530
---

A cloud service that helps us securely control access to AWS resources. IAM controls who is authenticated (signed in) and authorized (has permissions) to use resources. IAM allows us to manage users and their level of access to the aws console (dashboard).

## IAM Key terminology

*   **Users** End user such as people, employees of an organization etc. 
*   **Groups** A collection of users. Each user in the group will inherit the permissions of the group.
*   **Policies** Policies are made of documents, called policy documents. These documents are in JSON format and they give permissions as to what a user/group is able
to do.
*   **Roles** We create roles and assign them to AWS resources.

## IAM Best practices

*   During setup, root account is the first account created. It has complete admin access to all AWS resources.
*   New users have NO permissions by default.
*   New users are assigned an "access key ID" and a "Secret Key" when first created.
*   This key is meant for programmatic requests via AWS Cli and not used to access the console.
*   We can view the access key and the secret key once, if lost, we have to generate a new pair.
*   Always set up Multi-factor authentication especially for the root account.
*   We can create and customise password rotation policies.

*   **Create non-root User for administration duties** Don't use root user credentials to access AWS. Instead, create separate users for anyone who needs access to AWS account. Create an IAM user for yourself as well, give that user appropriate permissions, and use that IAM user for all administrative work.

*   **Use Groups to assign Permissions** Instead of defining permissions for individual IAM users, it's usually more convenient to create groups that relate to job functions (administrators, developers, accounting, etc). Next, define the relevant permissions for each group. Finally, assign IAM users to those groups.

*   **IAM Least Privilege** When creating any IAM policies, follow the standard security advice of granting least privilege, or granting only the permissions required to perform a task. Determine what users (and roles) need to do and then craft policies that allow them to perform only those tasks. 

*   **AWS Managed Policies** We can use AWS managed policies to give new users/employees the permissions they need to get started quickly. These policies are available by default, though with broad permissions, they are maintained and updated by AWS.

*   **Customer Managed Policies** A key advantage of using these policies is that we can view all managed policies in one place in the console. We can also view this information with a single AWS CLI or AWS API operation. Inline policies are policies that exist only on an IAM identity (user, group, or role). Managed policies are separate IAM resources that we can attach to multiple identities. 

*   **Review Permissions** AWS categorizes each service action into one of five access levels based on what each action does: List, Read, Write, Permissions management, or Tagging. We can use these access levels to determine which actions to include in the policy documents. 

*   **Enforcing Strong Password Policy** If we allow users to change their own passwords, require that they create strong passwords and that they rotate their passwords periodically. Define password requirements, such as minimum length, whether it requires non-alphabetic characters, how frequently it must be rotated, and so on.

*   **Use Roles for Apps running on EC2 Instances** Applications that run on an Amazon EC2 instance need credentials in order to access other AWS services. To provide credentials in a secure way, use should use IAM roles. A role is an entity that has its own set of permissions, but that isn't a user or group.

*   **Use Roles to Delegate Permissions** Don't share security credentials between accounts to allow users from another AWS account to access resources. Instead, use IAM roles. We can define a role that specifies what permissions the IAM users in the other account are allowed. We can also designate which AWS accounts have the IAM users that are allowed to assume the role.

*   **Embedding keys in source code** Access keys provide programmatic access to AWS. Do not embed access keys within unencrypted code. For applications that need access to AWS, configure the program to retrieve temporary security credentials using an IAM role. To allow individual programmatic access, create an IAM user with personal access keys.

*   **Use Policy Conditions for Extra Security** To the extent that it's practical, define the conditions under which IAM policies allow access to a resource. For example, we can specify a range of allowable IP addresses that a request must come from.

We can also specify that a request is allowed only within a specified date range or time range. We can also set conditions that require the use of SSL or MFA (multi-factor authentication). For example, us can require that a user has authenticated with an MFA device in order to be allowed to terminate (shutdown) an Amazon EC2 instance.

*   **Monitor Activity in the AWS Account** We can use logging features in AWS to review all actions the users have taken in their account and the resources that were used. The log files show the time and date of actions, the source IP for an action, and actions that failed due to inadequate permissions. 


##### Sources

- <https://goteleport.com/blog/aws-iam-in-laymans-terms/>
- <https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html>
- <https://docs.aws.amazon.com/IAM/latest/UserGuide/intro-structure.html>
- <https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html>
