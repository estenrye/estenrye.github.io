---
layout: post
title: Configuring AWS SSO with JumpCloud SAML Integration
date: 2022-09-09T09:14:00+00:00
categories:
  - homelab
  - aws
  - saml
  - jumpcloud
  - SSO
tags:
  - cd-homelab
  - automate-someday
---


# Prerequisites

- [Sign up](https://console.jumpcloud.com/signup) for a free JumpCloud Account
- [Sign up](https://portal.aws.amazon.com/billing/signup?nc2=h_ct&src=header_signup&redirect_url=https%3A%2F%2Faws.amazon.com%2Fregistration-confirmation#/start/email) for a free AWS Account
- After signing up for AWS, note your AWS Account Number.
- Define a Naming Convention for custom user attributes that will be used to
  grant AWS Role Permissions in JumpCloud.  The resulting values should be
  unique across AWS Accounts and Roles.
  - Example Naming Convention:  `AWS{{ AWS_Account_Number }}{{ AWS_Role_Name }}`
  - Example Attribute Name: `AWS673127022430ADM`

# Configuring JumpCloud SSO Application

1. Browse to the [JumpCloud Admin Console](https://console.jumpcloud.com/login/admin) and authenticate with Administrator credentials.
1. Click [SSO](https://console.jumpcloud.com/#/sso)
1. Click the [Plus button](https://console.jumpcloud.com/#/sso/choose) to configure a New SSO Application.
1. Search for `AWS`
1. Click Configure next to `Amazon Web Services (IAM)`.
1. On the `General Info` tab, provide a `Display Label`.
1. On the `SSO` tab, click `add attribute` under the `USER ATTRIBUTE MAPPING` section for each Account/Role custom user attribute.
  - For all attributes added, use `https://aws.amazon.com/SAML/Attributes/Role` for the 
    `Service Provider Attribute Name` and the Account/Role custom user attribute as the
    `JumpCloud Attribute Name`.
  - Example:
    - Service Provider Attribute Name: `https://aws.amazon.com/SAML/Attributes/Role`
    - JumpCloud Attribute Name: `AWS673127022430ADM`
1. On the `SSO` tab, under the `CONSTANT ATTRIBUTES` section:
  1. Delete the default `https://aws.amazon.com/SAML/Attributes/Role` attribute.
  2. Change the `https://aws.amazon.com/SAML/Attributes/SessionDuration` attribute to `43200` seconds (12 hours)
1. Click Activate to create the JumpCloud SSO Application
1. Select the newly created JumpCloud SSO Application and click `export metadata` to download the Identity Provider Metadata XML file.

# Configure a new Identity Provider in AWS Identity and Access Management (IAM)

1. Browse to the [AWS Admin Console](https://console.aws.amazon.com).
1. Log in with Administrative Credentials.
1. Under Services, select `IAM`.
1. In the [Identity and Access Management (IAM)](https://us-east-1.console.aws.amazon.com/iamv2/home?region=us-east-1#/home) dashboard,
   click [Identity providers](https://us-east-1.console.aws.amazon.com/iamv2/home?region=us-east-1#/identity_providers).
1. Click [Add provider](https://us-east-1.console.aws.amazon.com/iamv2/home?region=us-east-1#/identity_providers/create).
1. Choose `SAML` for the `Provider type`.
1. Choose a meaningful `Provider name`.
1. Click `Choose file` to browse to the downloaded Identity Provider Metadata XML file.
1. Add tags to facilitate searching and identification.
   - Example:
      - Tag Name: `IdentityProvider`
      - Tag Value: `JumpCloud`
1. Click Add Provider.
1. Click the newly created Identity Provider.
1. Copy the Identity Provider's ARN.

# Configure a new IAM Role to be assumed by SAML Identities in AWS Identity and Access Management (IAM)

1. Browse to the [AWS Admin Console](https://console.aws.amazon.com).
1. Log in with Administrative Credentials.
1. Under Services, select `IAM`.
1. In the [Identity and Access Management (IAM)](https://us-east-1.console.aws.amazon.com/iamv2/home?region=us-east-1#/home) dashboard,
   click [Roles](https://us-east-1.console.aws.amazon.com/iamv2/home?region=us-east-1#/roles).
1. Click [Create role](https://us-east-1.console.aws.amazon.com/iamv2/home?region=us-east-1#/roles/create?step=selectEntities).
1. Select `AWS account` for the `Trusted entity type`.
1. Specify the AWS Account Number.
1. Click Next.
1. Add required permissions to the role.
   - To create an Administrator role:
        - Search for the `AdministratorAccess` policy.
        - Click the checkbox next to the `AdministratorAccess` policy.
1. Click Next.
1. Provide a meaningful `Role name`.
1. Click `Edit` on `Step 1: Select trusted entities`.
1. For `Trusted entity type` choose `SAML 2.0 federation`.
1. For `SAML 2.0-based provider` select the Identity Provider you created in the previous section.
1. Select `Allow programmatic and AWS Management Console Access`.
1. Click Next.
1. Click Next.
1. Add tags to facilitate searching and identification.
   - Example:
      - Tag Name: `IdentityProvider`
      - Tag Value: `JumpCloud`
1. Click `Create role`.
1. Click `View role` on the green notification bar.
1. Copy the Role's ARN.

# Configure JumpCloud Group custom user attributes.

1. Browse to the [JumpCloud Admin Console](https://console.jumpcloud.com/login/admin) and authenticate with Administrator credentials.
1. Click [User Groups](https://console.jumpcloud.com/#/groups/user)
1. Click the [Plus button](https://console.jumpcloud.com/#/groups/user/new) to create a new group.
1. On the `Details` tab:
    1. Provide a meaningful name.
    1. Click `add new custom attribute` in the `Custom Attributes` section and select `String`
    1. Add Custom Attributes for each AWS IAM Role being granted to the Group.
        1. For `Attribute Name` provide an AWS Account/Role custom attribute value.
            - Example: `AWS673127022430ADM`
        1. For `Attribute Value` provide a comma delimited list containing both the AWS Role ARN and the AWS Identity Provider ARN.
            - Example: `arn:aws:iam::673127022430:role/JumpCloud-SSO-Administrator-Access,arn:aws:iam::673127022430:saml-provider/JumpCloud`
        1. Repeate for each AWS IAM Role.
1. On the `Users` tab, select which users should be granted AWS access.
1. On the `Applications` tab, grant the group access to the AWS SSO application.
1. Click Save

# Test your work by logging into the AWS Dashboard using JumpCloud SSO.

1. Browse to the [JumpCloud User Console](https://console.jumpcloud.com/userconsole#/)
1. Click the AWS tile.
1. Select the Role you want to assume.

# Configure saml2aws for command line access.

```bash
brew install saml2aws
saml2aws configure \
  --idp-account=ryezone-com \
  --url=https://sso.jumpcloud.com/saml2/aws-rzlbs \
  --idp-provider=JumpCloud \
  --mfa=WEBAUTHN \
  --profile=ryezone-com

saml2aws login \
  --idp-account=ryezone-com \
  --profile=ryezone-com \
  --region=us-east-2 \
  --role=arn:aws:iam::673127022430:role/JumpCloud-SSO-Administrator-Access \
  --cache-saml \
  --skip-prompt \
  --force
```