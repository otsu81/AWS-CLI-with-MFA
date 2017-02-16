# AWS-CLI-with-MFA
A best practice for IAM users is to use MFA. This is especially true when it comes to users in the admin group and should be a basic requirement. In a large multiple account setup where admin users are allowed to roleswitch, a compromised admin account without MFA can cause incredible damage. 

But MFA presents a problem. How do we enforce MFA on a legacy account with an established IAM account structure? And how do we do MFA for CLI?

Turns out it’s only possible to enforce MFA for CLI when assuming a role. This is useful knowledge because the same pattern can be used to enforce MFA is used for admin roles in child accounts.

Credit to Johannes Pelto-Piri for supplying policy templates.

## Assume role with CLI
Following instruction is for OS X. Windows and Linux is on TODO list.

When making a CLI call requiring a specific role the `--profile` option must always be appended. 
Example:

```
aws ec2 describe-hosts --region eu-west-1 --profile cli
```

In `~/.aws` there are two files of interest: `config` and `credentials`

`credentials` is automatically populated when running `aws configure` command, and contains the access key and secret access key. It can contain several profiles which is handy for dealing with different accounts. The config file can reference a profile in the credentials file, which is invoked when we make the `--profile [profilename]` call in the CLI.

### Example credentials file
```
[cli]
aws_access_key_id = AAAAACCCEEESSSSKEEEEEYYYY
aws_secret_access_key = SSSSSSUUUUUUUUPPPPEEEEERRRSEEEEEECRRRREEET
```

### Example config file
```
[profile cli]
role_arn = arn:aws:iam::<CHILD ACCOUNT NUMBER>:role/adminMFA-roleswitch
mfa_serial = arn:aws:iam::<MAIN ACCOUNT NUMBER>:mfa/adminUser
source_profile = cli
```

## Policies
This structure is based around two policies: the admin group MFA requirement policy, and the role policy in the child account. 

The **admin policy** allows an attached user admin privileges under the condition that there is MFA present. If there is no MFA attached, the user cannot do anything but go into IAM and add one or modify a previous one attached to the user. 

The **role policy** is necessary in every account where CLI access is needed, since it’s only possible to enforce MFA in CLI when assuming a role. The role policy has a trust to the main account and enforces MFA. 
