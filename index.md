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

## Basic Stack Workflow

## Stack Deploy

To deploy a new stack using the configuration file above:

```
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
$ cfn stack deploy IAMUsersGroupsAndPolicies.yaml --no-wait
```

You can also specify a canned stack policy (like you could do for S3 objects), or disable rollback on deployment failure (very useful when composing the CloudFormation with `cfn-tools`).

```
$ cfn stack deploy IAMUsersGroupsAndPolicies.yaml --canned-policy=DENY_ALL --on-error=NO_NOTHING
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

### Stack Tail and Stack Describe

You can use `stack describe` to inspect stack status:

```
$ cfn stack describe IAMUsersGroupsAndPolicies.yaml --detail=2
Stack ID: arn:aws:cloudformation:us-east-1:1234567890:stack/IAMUsersGroupsAndPolicies/39c11b10-d995-11e6-a8cf-503acac5c035
Name: IAMUsersGroupsAndPolicies
Description: AWS CloudFormation Sample Template IAM_Users_Groups_and_Policies.....
Status: CREATE_COMPLETE
Status Reason:
Created: 2017-01-13 13:36:02.447000+00:00
Capabilities: ['CAPABILITY_IAM']
Parameters:
  Password: ****
Outputs:
  SecretKey: EoBA@YjYM4a9zBC6oCznlvVznCZMmOmb@wj3foD5
  AccessKey: ABCDEFGHIJKLMN
Tags:
  project: Bob
Resources:
  Logical ID: Admins
    Type: AWS::IAM::UserToGroupAddition
    Physical ID: IAMUs-Admi-10RG67MMKA5D6
    Last Updated: 2017-01-13 13:36:46.865000+00:00
  ...
  Logical ID: Users
    Type: AWS::IAM::UserToGroupAddition
    Physical ID: IAMUs-User-1BN64OJ3I38G4
    Last Updated: 2017-01-13 13:36:46.576000+00:00  
```

Or use `stack tail` to retrive stack events:
```
$ cfn stack tail IAMUsersGroupsAndPolicies.yaml -t 30 -n 10
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
```

`-n` means retrive last 10 stack events and `-t 30` means only tail command will stop waiting for new events after 30 seconds.

### Stack Update

After modified CloudFormation template or stack parameters, you apply changes to the stack using `stack update` command.

First, lets apply a canned policy to the stack:

```
$ cfn stack update IAMUsersGroupsAndPolicies.yaml --canned-policy=DENY_ALL --use-previous-template
Region: us-east-1
Stack Name: IAMUsersGroupsAndPolicies
Template: https://s3.amazonaws.com/cloudformation-templates-us-east-1/IAM_Users_Groups_and_Policies.template
Updating stack...
Stack ID: arn:aws:cloudformation:us-east-1:1234567890:stack/IAMUsersGroupsAndPolicies/39c11b10-d995-11e6-a8cf-503acac5c035
01/13/17 13:36:49 - CREATE_COMPLETE - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - arn:aws:cloudformation:us-east-1:889515947644:stack/IAMUsersGroupsAndPolicies/39c11b10-d995-11e6-a8cf-503acac5c035
01/13/17 14:05:17 - UPDATE_IN_PROGRESS - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - User Initiated
01/13/17 14:05:22 - UPDATE_COMPLETE_CLEANUP_IN_PROGRESS - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - arn:aws:cloudformation:us-east-1:889515947644:stack/IAMUsersGroupsAndPolicies/39c11b10-d995-11e6-a8cf-503acac5c035
01/13/17 14:05:23 - UPDATE_COMPLETE - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - arn:aws:cloudformation:us-east-1:889515947644:stack/IAMUsersGroupsAndPolicies/39c11b10-d995-11e6-a8cf-503acac5c035
Stack update complete.
```

Then, modify the stack configuration file, change the password to another value:

```yaml
Stack:
  Region:               us-east-1
  StackName:            IAMUsersGroupsAndPolicies
  TemplateURL:          https://s3.amazonaws.com/cloudformation-templates-us-east-1/IAM_Users_Groups_and_Policies.template
  Capabilities:         [CAPABILITY_IAM]
  Parameters:
    Password:           123456
  Tags:
    project:            Bob
```

Then start stack update:

```
$ cfn stack update IAMUsersGroupsAndPolicies.yaml --use-previous-template
Region: us-east-1
Stack Name: IAMUsersGroupsAndPolicies
Template: https://s3.amazonaws.com/cloudformation-templates-us-east-1/IAM_Users_Groups_and_Policies.template
Updating stack...
Stack ID: arn:aws:cloudformation:us-east-1:1234567890:stack/IAMUsersGroupsAndPolicies/39c11b10-d995-11e6-a8cf-503acac5c035
01/13/17 14:05:23 - UPDATE_COMPLETE - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - arn:aws:cloudformation:us-east-1:889515947644:stack/IAMUsersGroupsAndPolicies/39c11b10-d995-11e6-a8cf-503acac5c035
01/13/17 14:09:41 - UPDATE_IN_PROGRESS - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - User Initiated
01/13/17 14:09:46 - UPDATE_FAILED - CFNUser(AWS::IAM::User) - Action denied by stack policy: Statement [#1] does not allow [Update:*] for resource [*];
01/13/17 14:09:47 - UPDATE_ROLLBACK_IN_PROGRESS - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - The following resource(s) failed to update: [CFNUser].
01/13/17 14:10:09 - UPDATE_COMPLETE - CFNUser(AWS::IAM::User) - IAMUsersGroupsAndPolicies-CFNUser-S9VVDYQL356B
01/13/17 14:10:12 - UPDATE_ROLLBACK_COMPLETE_CLEANUP_IN_PROGRESS - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - arn:aws:cloudformation:us-east-1:889515947644:stack/IAMUsersGroupsAndPolicies/39c11b10-d995-11e6-a8cf-503acac5c035
01/13/17 14:10:15 - UPDATE_ROLLBACK_COMPLETE - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - arn:aws:cloudformation:us-east-1:889515947644:stack/IAMUsersGroupsAndPolicies/39c11b10-d995-11e6-a8cf-503acac5c035
Waiter StackUpdateComplete failed: Waiter encountered a terminal failure state
```

Failed!  Thats because I just added a stack policy denys all stack resource changes.  I can overide this behaviour using `--override-policy`:

```
$ cfn stack update IAMUsersGroupsAndPolicies.yaml --use-previous-template --override-policy=ALLOW_ALL
Region: us-east-1
Stack Name: IAMUsersGroupsAndPolicies
Template: https://s3.amazonaws.com/cloudformation-templates-us-east-1/IAM_Users_Groups_and_Policies.template
Updating stack...
Overriding stack policy during update...
Stack ID: arn:aws:cloudformation:us-east-1:1234567890:stack/IAMUsersGroupsAndPolicies/39c11b10-d995-11e6-a8cf-503acac5c035
01/13/17 14:10:15 - UPDATE_ROLLBACK_COMPLETE - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - arn:aws:cloudformation:us-east-1:889515947644:stack/IAMUsersGroupsAndPolicies/39c11b10-d995-11e6-a8cf-503acac5c035
01/13/17 14:11:26 - UPDATE_IN_PROGRESS - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - User Initiated
01/13/17 14:11:31 - UPDATE_IN_PROGRESS - CFNUser(AWS::IAM::User) - IAMUsersGroupsAndPolicies-CFNUser-S9VVDYQL356B
01/13/17 14:11:31 - UPDATE_FAILED - CFNUser(AWS::IAM::User) - Password does not conform to the account password policy.
01/13/17 14:11:32 - UPDATE_ROLLBACK_IN_PROGRESS - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - The following resource(s) failed to update: [CFNUser].
01/13/17 14:11:40 - UPDATE_COMPLETE - CFNUser(AWS::IAM::User) - IAMUsersGroupsAndPolicies-CFNUser-S9VVDYQL356B
01/13/17 14:11:42 - UPDATE_ROLLBACK_COMPLETE_CLEANUP_IN_PROGRESS - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - arn:aws:cloudformation:us-east-1:889515947644:stack/IAMUsersGroupsAndPolicies/39c11b10-d995-11e6-a8cf-503acac5c035
01/13/17 14:11:44 - UPDATE_ROLLBACK_COMPLETE - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - arn:aws:cloudformation:us-east-1:889515947644:stack/IAMUsersGroupsAndPolicies/39c11b10-d995-11e6-a8cf-503acac5c035
Waiter StackUpdateComplete failed: Waiter encountered a terminal failure state
```

Failed again!  This time because the password is too simple for my account password policy (`123456` is a very bad password anyway), let's modifiy it to a strong password and try again:

```
$ cfn stack update IAMUsersGroupsAndPolicies.yaml --use-previous-template --override-policy=ALLOW_ALL
Region: us-east-1
Stack Name: IAMUsersGroupsAndPolicies
Template: https://s3.amazonaws.com/cloudformation-templates-us-east-1/IAM_Users_Groups_and_Policies.template
Updating stack...
Overriding stack policy during update...
Stack ID: arn:aws:cloudformation:us-east-1:1234567890:stack/IAMUsersGroupsAndPolicies/39c11b10-d995-11e6-a8cf-503acac5c035
01/13/17 14:11:44 - UPDATE_ROLLBACK_COMPLETE - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - arn:aws:cloudformation:us-east-1:889515947644:stack/IAMUsersGroupsAndPolicies/39c11b10-d995-11e6-a8cf-503acac5c035
01/13/17 14:13:46 - UPDATE_IN_PROGRESS - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - User Initiated
01/13/17 14:13:51 - UPDATE_IN_PROGRESS - CFNUser(AWS::IAM::User) - IAMUsersGroupsAndPolicies-CFNUser-S9VVDYQL356B
01/13/17 14:13:52 - UPDATE_COMPLETE - CFNUser(AWS::IAM::User) - IAMUsersGroupsAndPolicies-CFNUser-S9VVDYQL356B
01/13/17 14:13:57 - UPDATE_COMPLETE_CLEANUP_IN_PROGRESS - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - arn:aws:cloudformation:us-east-1:889515947644:stack/IAMUsersGroupsAndPolicies/39c11b10-d995-11e6-a8cf-503acac5c035
01/13/17 14:13:58 - UPDATE_COMPLETE - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - arn:aws:cloudformation:us-east-1:889515947644:stack/IAMUsersGroupsAndPolicies/39c11b10-d995-11e6-a8cf-503acac5c035
Stack update complete.
```
This time stack update is successful.

### Stack Delete

Finally, I delete the sample stack use:

```
$ cfn stack delete IAMUsersGroupsAndPolicies.yaml
Region: us-east-1
Stack Name: IAMUsersGroupsAndPolicies
Template: https://s3.amazonaws.com/cloudformation-templates-us-east-1/IAM_Users_Groups_and_Policies.template
Deleting stack...
Stack ID: arn:aws:cloudformation:us-east-1:1234567890:stack/IAMUsersGroupsAndPolicies/39c11b10-d995-11e6-a8cf-503acac5c035
01/13/17 14:13:58 - UPDATE_COMPLETE - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - arn:aws:cloudformation:us-east-1:889515947644:stack/IAMUsersGroupsAndPolicies/39c11b10-d995-11e6-a8cf-503acac5c035
01/13/17 14:17:52 - DELETE_IN_PROGRESS - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - User Initiated
01/13/17 14:17:56 - DELETE_IN_PROGRESS - Admins(AWS::IAM::UserToGroupAddition) - IAMUs-Admi-10RG67MMKA5D6
01/13/17 14:17:56 - DELETE_IN_PROGRESS - CFNAdminPolicies(AWS::IAM::Policy) - IAMUs-CFNA-1EO5Y72D9PUVK
01/13/17 14:17:56 - DELETE_IN_PROGRESS - Users(AWS::IAM::UserToGroupAddition) - IAMUs-User-1BN64OJ3I38G4
01/13/17 14:17:56 - DELETE_IN_PROGRESS - CFNKeys(AWS::IAM::AccessKey) - AKIAILIO2DDYIOWHWOQQ
01/13/17 14:17:56 - DELETE_IN_PROGRESS - CFNUserPolicies(AWS::IAM::Policy) - IAMUs-CFNU-UA1JEFKG9RJM
01/13/17 14:17:56 - DELETE_COMPLETE - Admins(AWS::IAM::UserToGroupAddition) - IAMUs-Admi-10RG67MMKA5D6
01/13/17 14:17:57 - DELETE_COMPLETE - CFNKeys(AWS::IAM::AccessKey) - AKIAILIO2DDYIOWHWOQQ
01/13/17 14:17:57 - DELETE_COMPLETE - Users(AWS::IAM::UserToGroupAddition) - IAMUs-User-1BN64OJ3I38G4
01/13/17 14:17:57 - DELETE_COMPLETE - CFNAdminPolicies(AWS::IAM::Policy) - IAMUs-CFNA-1EO5Y72D9PUVK
01/13/17 14:17:58 - DELETE_COMPLETE - CFNUserPolicies(AWS::IAM::Policy) - IAMUs-CFNU-UA1JEFKG9RJM
Stack delete complete.
```

## ChangeSet Workflow

ChangeSet allows you preview the blast radius before applying to an active stack.

### ChangeSet Create

A ChangeSet is created by apply update against an existing stack, or create a new stack in `REVIEW_IN_PROGRESS` state.

To create a new ChangeSet against the sample stack, use:

```
$ cfn changeset create IAMUsersGroupsAndPolicies.yaml
Region: us-east-1
Stack Name: IAMUsersGroupsAndPolicies
Template: https://s3.amazonaws.com/cloudformation-templates-us-east-1/IAM_Users_Groups_and_Policies.template
Creating change set...
ChangeSet Name: IAMUsersGroupsAndPolicies-ChangeSet-4d31327d
ChangeSet ARN: arn:aws:cloudformation:us-east-1:1234567890:changeSet/IAMUsersGroupsAndPolicies-ChangeSet-4d31327d/640ae4d6-c030-45eb-aff4-d02ab8cd4a54
ChangeSet creation complete.
```
Here, `awscfncli` generated an unqiue ChangeSet name since I didn't specify one.  Note the name in the output above:

    IAMUsersGroupsAndPolicies-ChangeSet-4d31327d

### Changeset Describe

Describe the change set to view all the stack resource changes:

```
$ cfn changeset describe IAMUsersGroupsAndPolicies.yaml IAMUsersGroupsAndPolicies-ChangeSet-4d31327d
Region: us-east-1
Stack Name: IAMUsersGroupsAndPolicies
Template: https://s3.amazonaws.com/cloudformation-templates-us-east-1/IAM_Users_Groups_and_Policies.template
ChangeSet Name: IAMUsersGroupsAndPolicies-ChangeSet-4d31327d
Execution Status: AVAILABLE
ChangeSet Status: CREATE_COMPLETE
Resource Changes:
  Action: Modify
  Logical Resource: CFNUser (AWS::IAM::User)
  Physical Resource: IAMUsersGroupsAndPolicies-CFNUser-V6D18EKNY0V6
  Replacement: False
  Scope: ['Properties']
```

### Changeset Execute

After review resource changes, execute the ChangeSet using:

```
$ cfn changeset execute IAMUsersGroupsAndPolicies.yaml IAMUsersGroupsAndPolicies-ChangeSet-4d31327d
Region: us-east-1
Stack Name: IAMUsersGroupsAndPolicies
Template: https://s3.amazonaws.com/cloudformation-templates-us-east-1/IAM_Users_Groups_and_Policies.template
01/13/17 16:39:59 - CREATE_COMPLETE - Admins(AWS::IAM::UserToGroupAddition) - IAMUs-Admi-4LNU55MI3DHX
01/13/17 16:40:00 - CREATE_IN_PROGRESS - CFNAdminPolicies(AWS::IAM::Policy) - Resource creation Initiated
01/13/17 16:40:00 - CREATE_COMPLETE - CFNAdminPolicies(AWS::IAM::Policy) - IAMUs-CFNA-1U6J4AUIT05MM
01/13/17 16:40:07 - CREATE_COMPLETE - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - arn:aws:cloudformation:us-east-1:1234567890:stack/IAMUsersGroupsAndPolicies/cd98d120-d9ae-11e6-9dbf-50fae9e50c9a
01/13/17 16:47:25 - UPDATE_IN_PROGRESS - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - User Initiated
01/13/17 16:47:33 - UPDATE_IN_PROGRESS - CFNUser(AWS::IAM::User) - IAMUsersGroupsAndPolicies-CFNUser-V6D18EKNY0V6
01/13/17 16:47:33 - UPDATE_COMPLETE - CFNUser(AWS::IAM::User) - IAMUsersGroupsAndPolicies-CFNUser-V6D18EKNY0V6
01/13/17 16:47:40 - UPDATE_COMPLETE_CLEANUP_IN_PROGRESS - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - arn:aws:cloudformation:us-east-1:889515947644:stack/IAMUsersGroupsAndPolicies/cd98d120-d9ae-11e6-9dbf-50fae9e50c9a
01/13/17 16:47:42 - UPDATE_COMPLETE - IAMUsersGroupsAndPolicies(AWS::CloudFormation::Stack) - arn:aws:cloudformation:us-east-1:889515947644:stack/IAMUsersGroupsAndPolicies/cd98d120-d9ae-11e6-9dbf-50fae9e50c9a
ChangSet execution complete.
```

This is very similar with stack update except policy override is not allowed.


## Stack Configuration Format

The stack configuration is a YAML file that specifies the runtime parameters of
a AWS CloudFormation template.

The definition of config scheme is as follows:

```yaml

type: map
mapping:
  "Stack":
    type: map
    mapping:
      "StackName":
        type: str
        required: yes
      "Region":
        type: str
        required: yes
      "TemplateBody":
        type: str
      "TemplateURL":
        type: str
      "Parameters":
        type: map
      "DisableRollback":
        type: bool
      "TimeoutInMinutes":
        type: int
      "NotificationARNs":
        type: seq
      "Capabilities":
        type: seq
      "ResourceTypes":
        type: seq
      "RoleARN":
        type: str
      "OnFailure":
        type: str
      "StackPolicyBody":
        type: str
      "StackPolicyURL":
        type: str
      "Tags":
        type: map
```

For more information about parameter values, please check to 
[boto3 CloudFormation CreateStack() document](http://boto3.readthedocs.io/en/latest/reference/services/cloudformation.html#CloudFormation.ServiceResource.create_stack). 
