# Tutorial

In this tutorial, I assume you are familiar with AWS and CloudFromation, and have already setup AWS creditinals for `boto3`.

## Stack Configuration

To start using `awscfncli` you must first create a stack configuration file.

A stack cofiguration file defines:

- Which region the stack is depolyed.
- Name of the tack.
- CloudFormation template (locally or remote, presumbally compiled from [troposphere](https://github.com/cloudtools/troposphere)).
- Other stack creation options.
- Stack parameters.
- Stack tags.

Unlike `aws-cli` which uses very verbose json, this configuration is defined using very simple `YAML`:


```yaml
Stack:
  Region:               us-east-1
  StackName:            IAMUsersGroupsAndPolicies
  TemplateURL:          https://s3.amazonaws.com/cloudformation-templates-us-east-1/IAM_Users_Groups_and_Policies.template
  Capabilities:         [CAPABILITY_IAM]
  Parameters:
    Password:           bob@180180180
  Tags:
    project:            Bob
```

This is the sample configuration file provided with `awscfncli`, you can download it from [here](https://github.com/Kotaimen/awscfncli/blob/master/samples/IAMUsersGroupsAndPolicies.yaml), I will use this file as sample file in this tutorial.
(If you already have a stack named `IAMUsersGroupsAndPolicies` in `us-east-1` region, please modify `StackName` in the configuration file.  This template does *not* create any billable resource.)

## Stack Workflow

## `stack deploy`

To deploy a new stack using the configuration file above:

```shell
$ cfn stack deploy IAMUsersGroupsAndPolicies.yaml

```
Here `stack` is a group command and `deploy` is a subcommand, after `cfn` starts executing the command, it will trace stack events and wait until stack deployment is complete, for this demo stack, it takes 1~2 minutes.  The output is color coded like the Console GUI which is not shown here.

```bash
Region: us-east-1
Stack Name: IAMUsersGroupsAndPolicies
Template: https://s3.amazonaws.com/cloudformation-templates-us-east-1/IAM_Users_Groups_and_Policies.template
Deploying stack...
Stack ID: arn:aws:cloudformation:us-east-1:1234567890:stack/IAMUsersGroupsAndPolicies/39c11b10-d995-11e6-a8cf-503acac5c035
01/13/17 13:36:02 - CREATE_IN_PROGRESS - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - User Initiated
01/13/17 13:36:06 - CREATE_IN_PROGRESS - CFNAdminGroup(AWS::IAM::Group)
01/13/17 13:36:06 - CREATE_IN_PROGRESS - CFNUser(AWS::IAM::User)
01/13/17 13:36:06 - CREATE_IN_PROGRESS - CFNUserGroup(AWS::IAM::Group)
01/13/17 13:36:07 - CREATE_IN_PROGRESS - CFNUser(AWS::IAM::User) - Resource creation Initiated
01/13/17 13:36:07 - CREATE_IN_PROGRESS - CFNUserGroup(AWS::IAM::Group) - Resource creation Initiated
01/13/17 13:36:07 - CREATE_IN_PROGRESS - CFNAdminGroup(AWS::IAM::Group) - Resource creation Initiated
01/13/17 13:36:42 - CREATE_COMPLETE - CFNAdminGroup(AWS::IAM::Group) - IAMUsersGroupsAndPolicies-CFNAdminGroup-1NV3IS12UKA1Z
01/13/17 13:36:42 - CREATE_COMPLETE - CFNUserGroup(AWS::IAM::Group) - IAMUsersGroupsAndPolicies-CFNUserGroup-1FRQPT20MJFGM
01/13/17 13:36:42 - CREATE_COMPLETE - CFNUser(AWS::IAM::User) - IAMUsersGroupsAndPolicies-CFNUser-S9VVDYQL356B
01/13/17 13:36:45 - CREATE_IN_PROGRESS - CFNKeys(AWS::IAM::AccessKey)
01/13/17 13:36:45 - CREATE_IN_PROGRESS - CFNKeys(AWS::IAM::AccessKey) - Resource creation Initiated
01/13/17 13:36:45 - CREATE_IN_PROGRESS - CFNUserPolicies(AWS::IAM::Policy)
01/13/17 13:36:45 - CREATE_IN_PROGRESS - Users(AWS::IAM::UserToGroupAddition)
01/13/17 13:36:45 - CREATE_IN_PROGRESS - Admins(AWS::IAM::UserToGroupAddition)
01/13/17 13:36:46 - CREATE_COMPLETE - CFNKeys(AWS::IAM::AccessKey) - AKIAILIO2DDYIOWHWOQQ
01/13/17 13:36:46 - CREATE_IN_PROGRESS - Users(AWS::IAM::UserToGroupAddition) - Resource creation Initiated
01/13/17 13:36:46 - CREATE_IN_PROGRESS - CFNAdminPolicies(AWS::IAM::Policy)
01/13/17 13:36:46 - CREATE_IN_PROGRESS - Admins(AWS::IAM::UserToGroupAddition) - Resource creation Initiated
01/13/17 13:36:46 - CREATE_COMPLETE - Users(AWS::IAM::UserToGroupAddition) - IAMUs-User-1BN64OJ3I38G4
01/13/17 13:36:46 - CREATE_IN_PROGRESS - CFNUserPolicies(AWS::IAM::Policy) - Resource creation Initiated
01/13/17 13:36:46 - CREATE_COMPLETE - Admins(AWS::IAM::UserToGroupAddition) - IAMUs-Admi-10RG67MMKA5D6
01/13/17 13:36:47 - CREATE_IN_PROGRESS - CFNAdminPolicies(AWS::IAM::Policy) - Resource creation Initiated
01/13/17 13:36:47 - CREATE_COMPLETE - CFNUserPolicies(AWS::IAM::Policy) - IAMUs-CFNU-UA1JEFKG9RJM
01/13/17 13:36:47 - CREATE_COMPLETE - CFNAdminPolicies(AWS::IAM::Policy) - IAMUs-CFNA-1EO5Y72D9PUVK
01/13/17 13:36:49 - CREATE_COMPLETE - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - arn:aws:cloudformation:us-east-1:889515947644:stack/IAMUsersGroupsAndPolicies/39c11b10-d995-11e6-a8cf-503acac5c035
Stack deployment complete.
```

After "Stack ID" is displayed the stack deployment process is already started, your can press `CTRL+C anytime` to stop tailing stack events.

If you don't want wait until stack deployment complete, use `--no-wait` option.

```
cfn stack deploy IAMUsersGroupsAndPolicies.yaml --no-wait
```

You can also specify a canned stack policy (like you could do for S3 objects), or disable rollback on deployment failure (very useful when composing the CloudFormation with `cfn-tools`).

```
cfn stack deploy IAMUsersGroupsAndPolicies.yaml --canned-policy=DENY_ALL --on-error=NO_NOTHING
```

#### Supported Canned Policy

`ALLOW_ALL`

```json
{
  "Statement" : [
    {
      "Effect" : "Allow",
      "Action" : "Update:*",
      "Principal": "*",
      "Resource" : "*"
    }
  ]
}
```

`ALLOW_MODIFY`

```json
{
  "Statement" : [
    {
      "Effect" : "Deny",
       "Action" : ["Update:Replace", "Update:Delete"],
      "Principal": "*",
      "Resource" : "*"
    }
  ]
}
```

`DENY_DELETE`

```json
{
  "Statement" : [
    {
      "Effect" : "Deny",
      "Action" : "Update:Delete",
      "Principal": "*",
      "Resource" : "*"
    }
  ]
}
```

`DENY_ALL`

```json
{
  "Statement" : [
    {
      "Effect" : "Deny",
      "Action" : "Update:*",
      "Principal": "*",
      "Resource" : "*"
    }
  ]
}
```

### `stack tail` and `stack describe `

### `stack update`

### `stack delete`

## ChangeSet Workflow

### `changeset create`

### `changeset describe`

### `changeset execute`
