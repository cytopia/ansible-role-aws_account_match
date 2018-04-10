# Ansible AWS account match


The `aws-account-match` role lets you define one or more AWS accounts and ensures your playbook
is only provisioned to those accounts.

Include this role at the very top of any AWS relatated playbooks and it will act as a safeguard
by immediately aborting the whole play in case you are logged in with the wrong AWS account.

Boto profiles and environment variables are both supported.

---


## Variables

**Note:** (XOR)

* Either `aws_account_match_allowed_account` or `aws_account_match_allowed_accounts` must be specified.
* Only one of `aws_account_match_allowed_account` or `aws_account_match_allowed_accounts` can be specified.

| Name                                 | Type                      | Description               |
|--------------------------------------|---------------------------|---------------------------|
| `aws_account_match_allowed_account`  | `int` or `string`         | Single allowed account    |
| `aws_account_match_allowed_accounts` | `list`                    | Multiple allowed accounts |
| `aws_account_match_profile`          | `string`                  | Optional boto profile of current login |


## Requirements

* [aws-cli](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) (`pip install awscli`)


## Usage

##### playbook.yml

```yml
---
- hosts: all
  roles:
    - aws-account-match  # <- make sure it is the first
    - aws-do-this
    - aws-do-that
    - aws-more-work
  tags:
    - aws
```
##### group_vars/all.yml

```yml
---
# Allow two AWS accounts
aws_account_match_allowed_accounts:
  - 123456789
  - 987654321

# Ensure to have boto profile optionally parsed via cmd args
aws_account_match_profile: "{{ profile | default('') }}"
```

#### Run

The `testing` boto profile has a login for account `555444333222`, which is not part of the allowed
accounts. In case this is run, it will immediately abort the whole play:

```bash
$ ansible-playbook playbook.yml -e profile=testing
```

```
TASK [aws-account-match : ensure current account matched any of all allowed accounts] **************************
fatal: [infrastructure]: FAILED! => {
    "assertion": "_aws_account_match_number.stdout | string in aws_account_match_allowed_accounts | map('string') | list",
    "changed": false,
    "evaluated_to": false,
    "msg": "Currently logged in account (555444333222) not in allowed accounts (123456789, 987654321)."
}
```

