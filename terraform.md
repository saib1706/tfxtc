# TF CloudInfra

## Table of Contents

* [Child Account Setup](#child-account-setup)
* [Documentation](./docs/README.md)
* [Terraform workflow process for module deployment](#terraform-workflow-process-for-deployment)
  * [Directory Layout](#directory-layout)
  * [Development Workflow](#development-workflow)
* [Known Issues and Bugs](./docs/known-issues.md)

## Child Account Setup
Follow the steps outlined in the [child account setup documentation](./docs/child-account-setup.md) to setup a child account.
## Terraform Workflow Process for Module Deployment

### Directory Layout

1. Keep the hierarchy shallow, avoid many layers of module references since bubbling up outputs can be problematic.

2. Below is an example of the structure for the AWS resources required for deployment.
The env directory contains a series of environments that will contain different projects and teams.  

3. The env/cellbio/ directory is considered the root module. All root modules will reference the modules directory.
Each `.tf` configuration should be named for the service that it intends to create.

4. Place all variables (input) within the `variables.tf` file

The purpose of this structure is to avoid duplicating code, as well as providing a way to run `terraform apply` within discrete directories, while also supporting development of terraform configurations with terraform workspaces (OSS)

```
├── env
│   └── dev
│       └── cellbio
│           ├── global
│           │   ├── main.tf
│           │   └── modules.tf
│           └── us-east-1
│               ├── main.tf
│               └── modules.tf
└── modules
    └── child_iam
        └── iam.tf
```
### Development workflow
1. There exists a mainline branch that is stable and builds.
1. For every new feature a `feature branch` should be created. If your feature is tied to a Jira issue, follow the naming convention of naming the branch `<first and last initial>/<issue number>`. For example `jn/DEV-1`
1. Developers should open a Pull Requests (PR) as early as possible.
   * The PR should contain a description of the changes, previous behavior, and how the change has been tested.
   Each PR should either be in the following state, with its state reflective in the PR's title:

   * Format of PR title: "[STATE] - [issue-#] - Title"
     - [WIP] - This state indicates that work is still in progress and no review is needed yet.
     - [REVIEW] - This state indicates that you would like a code review and feedback on your code.
     - [READY] - This state indicates that the PR is ready to merge. Ready to merge means that another team member has approved the PR. A requirement of this state is that a team member other than you must do the merge, not yourself.

   * This makes it so that whoever is looking at the PRs knows exactly what the status of the PR is, rather than clicking into the PR.
1. The feature being developed should be tested and verified in a development environment. If the feature being developed is defined in terraform, and if the feature is workspace "aware", it should be tested in the developer's unique terraform workspace. Otherwise testing in the default workspace should suffice. If terraform is involved, please take a snippet of the `terraform plan` output and paste it INSIDE the PR itself. This allows for other team members to take a second look at the code and provide feedback.
1. Developer should rebase the master branch onto their feature branch due to the possibility that other team member's feature branches could have been merged into the mainline branch. This is done via the following steps:
   1. `git checkout master && git pull --rebase origin master`
   1. `git checkout <your-branch> && git rebase master`
1. Once validation of functionality is complete and proof (tests, cli output, screenshots, etc.) has been submitted back to the existing PR for other to review, then the PR should be updated with a `READY`.
1. A developer can never merge their own `Pull Request`.
1. Once reviewed by other team members it can be merged.
1. After the merge to master, the developer should pull the master branch on their local workstation and create a new `feature branch`.
   * `git checkout master && git pull --rebase origin master`

### How to handle secrets within AWS
We are going to be leveraging AWS Systems Manager Parameter Store for the following scenarios:
- You or a service needs to store/consume a secret value
    - Hiearchy of secret path should be `<env>/<component>/<secret>`
- You need to send a team member sensitive information. In this scenario, you would store the info into AWS Systems Manager Parameter Store, and have the other person pull down that value.
    - Hiearchy of secret path should be `<service>/<secret>` if it's being used in environment you own.
    - Hiearchy of secret path should be `<account>/<service>/<secret>` if it's being used/stored in a different account.


AWS CLI example to push up secrets
```
aws ssm put-parameter --name <name_of_secret> --type SecureString --value <value_of_secret> --region <region>
```

AWS CLI example to pull down secrets
```
aws ssm get-parameter --name <name_of_secret> --with-decryption --region <region> --output text --query Parameter.Value
```
