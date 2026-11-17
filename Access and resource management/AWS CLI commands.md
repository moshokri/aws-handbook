# 1.1 AWS CLI commands for access and resource management

- [Setup and profiles](#setup-and-profiles)
- [Identity and credentials](#identity-and-credentials)
- [AWS Organizations](#aws-organizations)
- [AWS Control Tower](#aws-control-tower)
- [AWS IAM](#aws-iam)
- [IAM Access Analyzer](#iam-access-analyzer)
- [AWS STS](#aws-sts)
- [IAM Identity Center](#iam-identity-center)
- [Regions and Availability Zones](#regions-and-availability-zones)
- [Tags and resource groups](#tags-and-resource-groups)
- [Billing and cost management](#billing-and-cost-management)
- [Recommendations and quotas](#recommendations-and-quotas)
- [Useful global options](#useful-global-options)

Refer to [Access and resource management](<Access and resource management.md>) for the concepts.

---

## Setup and profiles

- The AWS CLI v2 is the current version. AWS CloudShell has it already.

```bash
aws --version

# Configure a profile with access keys (avoid long-term keys for people)
aws configure --profile dev

# Configure a profile for IAM Identity Center (SSO)
aws configure sso --profile dev

# Sign in and sign out with IAM Identity Center
aws sso login --profile dev
aws sso logout

# Show the configuration that the CLI uses
aws configure list --profile dev
aws configure list-profiles

# Set one value in the configuration
aws configure set region eu-west-1 --profile dev
```

- The CLI keeps configuration in `~/.aws/config` and keys in `~/.aws/credentials`.
- Use the environment variable `AWS_PROFILE` to select a profile for all commands.

```bash
export AWS_PROFILE=dev
export AWS_REGION=eu-west-1
```

## Identity and credentials

```bash
# Show the account, user ID, and ARN of the caller
aws sts get-caller-identity

# Show only the account ID
aws sts get-caller-identity --query Account --output text
```

## AWS Organizations

```bash
# Organization and root
aws organizations describe-organization
aws organizations list-roots

# Accounts
aws organizations list-accounts
aws organizations create-account --email dev@example.com --account-name "App-dev"
aws organizations describe-create-account-status --create-account-request-id <request-id>
aws organizations move-account --account-id <account-id> \
  --source-parent-id <root-or-ou-id> --destination-parent-id <ou-id>

# Organizational units
aws organizations list-organizational-units-for-parent --parent-id <root-id>
aws organizations create-organizational-unit --parent-id <root-id> --name Workloads
aws organizations list-accounts-for-parent --parent-id <ou-id>

# Turn on a policy type (SERVICE_CONTROL_POLICY, RESOURCE_CONTROL_POLICY, TAG_POLICY, BACKUP_POLICY, DECLARATIVE_POLICY_EC2)
aws organizations enable-policy-type --root-id <root-id> --policy-type SERVICE_CONTROL_POLICY

# Service control policies
aws organizations list-policies --filter SERVICE_CONTROL_POLICY
aws organizations create-policy --name DenyLeaveOrg --type SERVICE_CONTROL_POLICY \
  --description "Deny leave organization" --content file://scp.json
aws organizations attach-policy --policy-id <policy-id> --target-id <ou-or-account-id>
aws organizations detach-policy --policy-id <policy-id> --target-id <ou-or-account-id>
aws organizations list-policies-for-target --target-id <ou-or-account-id> --filter SERVICE_CONTROL_POLICY

# Delegated administrator
aws organizations register-delegated-administrator --account-id <account-id> \
  --service-principal guardduty.amazonaws.com
aws organizations list-delegated-administrators
```

Example SCP (`scp.json`):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    { "Effect": "Deny", "Action": "organizations:LeaveOrganization", "Resource": "*" }
  ]
}
```

## AWS Control Tower

```bash
# Landing zones
aws controltower list-landing-zones
aws controltower get-landing-zone --landing-zone-identifier <landing-zone-arn>

# Controls on an OU
aws controltower list-enabled-controls --target-identifier <ou-arn>
aws controltower enable-control --control-identifier <control-arn> --target-identifier <ou-arn>
aws controltower disable-control --control-identifier <control-arn> --target-identifier <ou-arn>
aws controltower get-control-operation --operation-identifier <operation-id>
```

- Account Factory uses AWS Service Catalog. Use `aws servicecatalog provision-product` to create an account from the CLI.

## AWS IAM

### Users and groups

```bash
aws iam list-users
aws iam create-user --user-name alice
aws iam create-group --group-name Developers
aws iam add-user-to-group --user-name alice --group-name Developers
aws iam list-groups-for-user --user-name alice

# Access keys (rotate them, or better, do not use them)
aws iam create-access-key --user-name alice
aws iam list-access-keys --user-name alice
aws iam update-access-key --user-name alice --access-key-id <key-id> --status Inactive
aws iam delete-access-key --user-name alice --access-key-id <key-id>

# MFA devices
aws iam list-mfa-devices --user-name alice
```

### Roles

```bash
aws iam list-roles
aws iam create-role --role-name AppRole --assume-role-policy-document file://trust.json
aws iam get-role --role-name AppRole
aws iam update-assume-role-policy --role-name AppRole --policy-document file://trust.json
aws iam delete-role --role-name AppRole

# Instance profile for EC2
aws iam create-instance-profile --instance-profile-name AppProfile
aws iam add-role-to-instance-profile --instance-profile-name AppProfile --role-name AppRole
```

Example trust policy (`trust.json`):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "ec2.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

### Policies

```bash
# Managed policies
aws iam list-policies --scope Local
aws iam create-policy --policy-name S3ReadOnlyApp --policy-document file://policy.json
aws iam attach-role-policy --role-name AppRole --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
aws iam detach-role-policy --role-name AppRole --policy-arn <policy-arn>
aws iam list-attached-role-policies --role-name AppRole

# Policy versions (a managed policy keeps a maximum of 5 versions)
aws iam create-policy-version --policy-arn <policy-arn> --policy-document file://policy.json --set-as-default
aws iam get-policy-version --policy-arn <policy-arn> --version-id v2

# Inline policies
aws iam put-role-policy --role-name AppRole --policy-name Inline --policy-document file://policy.json
aws iam list-role-policies --role-name AppRole

# Permissions boundary
aws iam put-role-permissions-boundary --role-name AppRole --permissions-boundary <policy-arn>
```

### Test and audit

```bash
# Simulate a policy before you attach it
aws iam simulate-principal-policy --policy-source-arn <role-arn> \
  --action-names s3:GetObject --resource-arns arn:aws:s3:::my-bucket/*

# Account summary and password policy
aws iam get-account-summary
aws iam get-account-password-policy

# Credential report (users, keys, MFA, last use)
aws iam generate-credential-report
aws iam get-credential-report --query Content --output text | base64 --decode

# Last access to services (find unused permissions)
aws iam generate-service-last-accessed-details --arn <role-arn>
aws iam get-service-last-accessed-details --job-id <job-id>
```

## IAM Access Analyzer

```bash
# Analyzer for external access (type ACCOUNT or ORGANIZATION)
aws accessanalyzer create-analyzer --analyzer-name org-analyzer --type ORGANIZATION
aws accessanalyzer list-analyzers
aws accessanalyzer list-findings-v2 --analyzer-arn <analyzer-arn>

# Analyzer for unused access
aws accessanalyzer create-analyzer --analyzer-name unused --type ACCOUNT_UNUSED_ACCESS

# Validate a policy against grammar and good practices
aws accessanalyzer validate-policy --policy-type IDENTITY_POLICY --policy-document file://policy.json

# Generate a policy from CloudTrail activity
aws accessanalyzer start-policy-generation \
  --policy-generation-details principalArn=<role-arn> \
  --cloud-trail-details file://cloudtrail-details.json
aws accessanalyzer get-generated-policy --job-id <job-id>
```

## AWS STS

```bash
# Assume a role (also cross-account)
aws sts assume-role --role-arn arn:aws:iam::<account-id>:role/AppRole \
  --role-session-name my-session --duration-seconds 3600

# Assume a role with MFA
aws sts assume-role --role-arn <role-arn> --role-session-name s1 \
  --serial-number <mfa-arn> --token-code 123456

# Assume a role with an OIDC token (for example, from CI/CD)
aws sts assume-role-with-web-identity --role-arn <role-arn> \
  --role-session-name ci --web-identity-token file://token.jwt

# Decode an "encoded authorization failure" message
aws sts decode-authorization-message --encoded-message <message>
```

- To use a role automatically, add it to a profile in `~/.aws/config`:

```ini
[profile prod-admin]
role_arn = arn:aws:iam::111122223333:role/Admin
source_profile = dev
mfa_serial = arn:aws:iam::444455556666:mfa/alice
region = eu-west-1
```

## IAM Identity Center

```bash
# Instance ARN and identity store ID
aws sso-admin list-instances

# Users and groups in the Identity Center directory
aws identitystore list-users --identity-store-id <store-id>
aws identitystore list-groups --identity-store-id <store-id>
aws identitystore create-group --identity-store-id <store-id> --display-name Developers

# Permission sets
aws sso-admin create-permission-set --instance-arn <instance-arn> \
  --name ReadOnly --session-duration PT8H
aws sso-admin attach-managed-policy-to-permission-set --instance-arn <instance-arn> \
  --permission-set-arn <ps-arn> --managed-policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
aws sso-admin list-permission-sets --instance-arn <instance-arn>

# Assign a group to an account with a permission set
aws sso-admin create-account-assignment --instance-arn <instance-arn> \
  --target-id <account-id> --target-type AWS_ACCOUNT \
  --permission-set-arn <ps-arn> --principal-type GROUP --principal-id <group-id>

# Push changes of a permission set to the accounts
aws sso-admin provision-permission-set --instance-arn <instance-arn> \
  --permission-set-arn <ps-arn> --target-type ALL_PROVISIONED_ACCOUNTS
```

## Regions and Availability Zones

```bash
aws ec2 describe-regions --output table
aws ec2 describe-regions --all-regions   # Also shows Regions that are not turned on
aws ec2 describe-availability-zones --region eu-west-1

# Local Zones and Wavelength Zones
aws ec2 describe-availability-zones --all-availability-zones \
  --filters Name=zone-type,Values=local-zone
aws ec2 modify-availability-zone-group --group-name <zone-group> --opt-in-status opted-in

# Turn on an opt-in Region for an account
aws account enable-region --region-name ap-east-1
aws account list-regions
```

## Tags and resource groups

```bash
# Tag and untag resources of many services with one API
aws resourcegroupstaggingapi tag-resources --resource-arn-list <arn1> <arn2> \
  --tags Environment=prod,CostCenter=1234
aws resourcegroupstaggingapi untag-resources --resource-arn-list <arn> --tag-keys CostCenter

# Find resources by tag
aws resourcegroupstaggingapi get-resources --tag-filters Key=Environment,Values=prod
aws resourcegroupstaggingapi get-tag-keys
aws resourcegroupstaggingapi get-tag-values --key Environment

# Resource groups
aws resource-groups create-group --name prod-resources \
  --resource-query '{"Type":"TAG_FILTERS_1_0","Query":"{\"ResourceTypeFilters\":[\"AWS::AllSupported\"],\"TagFilters\":[{\"Key\":\"Environment\",\"Values\":[\"prod\"]}]}"}'
aws resource-groups list-groups
aws resource-groups list-group-resources --group-name prod-resources

# Activate cost allocation tags
aws ce update-cost-allocation-tags-status \
  --cost-allocation-tags-status TagKey=CostCenter,Status=Active
```

## Billing and cost management

```bash
# Cost Explorer: monthly cost by service
aws ce get-cost-and-usage \
  --time-period Start=2026-10-01,End=2026-11-01 \
  --granularity MONTHLY --metrics UnblendedCost \
  --group-by Type=DIMENSION,Key=SERVICE

# Cost forecast
aws ce get-cost-forecast --time-period Start=2026-11-16,End=2026-12-01 \
  --granularity MONTHLY --metric UNBLENDED_COST

# AWS Budgets
aws budgets describe-budgets --account-id <account-id>
aws budgets create-budget --account-id <account-id> \
  --budget file://budget.json --notifications-with-subscribers file://notifications.json

# Cost Anomaly Detection
aws ce get-anomaly-monitors
aws ce get-anomalies --date-interval StartDate=2026-11-01,EndDate=2026-11-16

# Savings Plans recommendations
aws ce get-savings-plans-purchase-recommendation \
  --savings-plans-type COMPUTE_SP --term-in-years ONE_YEAR \
  --payment-option NO_UPFRONT --lookback-period-in-days THIRTY_DAYS

# Data Exports (CUR 2.0)
aws bcm-data-exports list-exports
```

Example budget (`budget.json`):

```json
{
  "BudgetName": "monthly-100",
  "BudgetLimit": { "Amount": "100", "Unit": "USD" },
  "TimeUnit": "MONTHLY",
  "BudgetType": "COST"
}
```

## Recommendations and quotas

```bash
# Trusted Advisor (Business, Enterprise On-Ramp, or Enterprise Support)
aws trustedadvisor list-checks
aws trustedadvisor list-recommendations

# Compute Optimizer
aws compute-optimizer update-enrollment-status --status Active --include-member-accounts
aws compute-optimizer get-ec2-instance-recommendations
aws compute-optimizer get-lambda-function-recommendations

# Cost Optimization Hub
aws cost-optimization-hub list-recommendations

# Service Quotas
aws service-quotas list-service-quotas --service-code ec2
aws service-quotas get-service-quota --service-code ec2 --quota-code L-1216C47A
aws service-quotas request-service-quota-increase --service-code ec2 \
  --quota-code L-1216C47A --desired-value 256
aws service-quotas list-requested-service-quota-change-history --service-code ec2
```

## Useful global options

| Option | Use |
|---|---|
| `--profile <name>` | Use a named profile. |
| `--region <code>` | Send the request to a Region. |
| `--output json\|yaml\|text\|table` | Set the output format. |
| `--query <JMESPath>` | Filter the output on the client. |
| `--filters` | Filter the output on the server (some services). |
| `--no-cli-pager` | Do not open the output in a pager. |
| `--dry-run` | Check permissions without a change (EC2 only). |
| `--cli-input-json file://x.json` | Read the parameters from a file. |
| `--generate-cli-skeleton` | Show a template for `--cli-input-json`. |
| `--debug` | Show the full request and response. |

```bash
# Example: names of all roles, as text
aws iam list-roles --query 'Roles[].RoleName' --output text

# Example: help for a command
aws iam create-role help
```
