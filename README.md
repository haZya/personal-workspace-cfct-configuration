# Personal Workspace CfCT Configuration

This repository contains the Customizations for AWS Control Tower (CfCT) configuration package for the `hazya.dev` workload accounts.

It installs GitHub Actions OIDC access into workload accounts so application repositories can deploy AWS resources through GitHub Actions without long-lived AWS access keys.

## Structure

```text
manifest.yaml
templates/
  github-actions-oidc-provider.template
  github-actions-role.template
```

## Target Accounts

The current manifest targets these AWS Control Tower OUs and accounts:

```text
Workloads:Production:hazya.dev
Workloads:PreProduction:hazya.dev
Staging
Preview
```

## Resources Created

The manifest deploys these StackSets:

```text
github-actions-oidc-provider-workloads
github-actions-role-production
github-actions-role-staging
github-actions-role-preview
```

The OIDC provider StackSet creates one IAM OIDC provider per workload account:

```text
token.actions.githubusercontent.com
```

The role StackSets create an IAM role named:

```text
GitHubActionsRole
```

The role currently attaches:

```text
arn:aws:iam::aws:policy/AdministratorAccess
```

This is intentionally broad for the initial OpenNext/Next.js setup. Replace it with least-privilege permissions after the deployed AWS resources are known.

## GitHub Trust Model

The role template uses exact GitHub OIDC subjects through the `GitHubSubjects` parameter.

Example production subject:

```text
repo:haZya/hazya.dev:environment:production
```

Example staging subject:

```text
repo:haZya/hazya.dev:environment:staging
```

Example preview subject:

```text
repo:haZya/hazya.dev:environment:preview
```

To trust multiple repositories for the same account role, use a comma-separated list:

```yaml
- parameter_key: GitHubSubjects
  parameter_value: repo:haZya/hazya.dev:environment:production,repo:haZya/another-repo:environment:production
```

Prefer exact subjects over wildcards such as `repo:haZya/*`.

## CfCT GitHub Source Setup

If CfCT is configured to use GitHub as its CodePipeline source, create an AWS CodeConnections connection in the AWS management account and Control Tower home Region.

Use these CfCT installer parameters:

```text
CodePipelineSource = GitHub (via Code Connection)
CodeConnection = <CODECONNECTION_ARN>
GitHubOwnerName = haZya
GitHubRepositoryName = <THIS_REPOSITORY_NAME>
GitHubBranchName = main
```

The GitHub repository does not need to be public. Private repositories work if the AWS GitHub app is authorized for the repo.

## Before Deployment

Review `manifest.yaml` and replace any remaining placeholders:

```text
<GITHUB_REPOSITORY>
```

Confirm these values match AWS Organizations and GitHub exactly:

```text
OU paths
Account names
GitHub owner
GitHub repository names
GitHub environment names
```

## GitHub Actions Usage

In an application repository, configure AWS credentials with GitHub OIDC.

Example:

```yaml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::<ACCOUNT_ID>:role/GitHubActionsRole
    aws-region: us-east-1
```

The workflow must run against the GitHub environment that matches the trusted subject, such as `production`, `staging`, or `preview`.

## Installer Template

[customizations-for-aws-control-tower.template](https://github.com/aws-solutions/aws-control-tower-customizations/blob/main/customizations-for-aws-control-tower.template) is the one-time CfCT solution installer or upgrade template for the AWS management account.
