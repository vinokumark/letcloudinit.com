---
title: Vault IAM Authentication
author: vinoth
layout: post
image: assets/images/blog/post-6.jpg
tags: []
---

Hope you aware of the [Hashicorp Vault](http://https://www.vaultproject.io/).  There are different auth methods available to connect the vault server or cluster.

One of the methods is AWS IAM authentication. We will see explore here the AWS IAM auth method with a simple dev vault server.

**Vault server**

We used Vault dev as a standalone server. For vault installation, please refer the vault [document](https://www.vaultproject.io/docs/install).

**config.hcl**

```
ui = true

#mlock = true
disable_mlock = true

listener "tcp" {
  address     = "172.31.32.239:8200"
  tls_disable = 1
}

 api_addr = "http://your-instance-ip:8200"

```
 Before going to launch the vault server. Create a simple IAM role with a simple policy, you can edit based on your requirement.

# **Recommended Vault IAM Policy**

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "iam:GetInstanceProfile",
        "iam:GetUser",
        "iam:GetRole"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": ["sts:AssumeRole"],
      "Resource": ["arn:aws:iam::<AccountId>:role/<VaultRole>"]
    },
    {
      "Sid": "ManageOwnAccessKeys",
      "Effect": "Allow",
      "Action": [
        "iam:CreateAccessKey",
        "iam:DeleteAccessKey",
        "iam:GetAccessKeyLastUsed",
        "iam:GetUser",
        "iam:ListAccessKeys",
        "iam:UpdateAccessKey"
      ],
      "Resource": "arn:aws:iam::*:user/${aws:username}"
    }
  ]
}
	
```


**Initiated the vault server**

vault server -dev -config=config.hcl

Log in with your root token and enable the AWS auth method

```
vault auth enable aws
```

**Create  a policy with capabilities**

```
vault policy write "example-policy" -<<EOF
path "secret/data/*" {
  capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}
EOF
```
**Map your policy with a secific role**

```
vault write auth/aws/role/vaul-test auth_type=iam \
              bound_iam_principal_arn=arn:aws:iam::562853193375:role/vault-test policies=example-policy max_ttl=500h
```
																								
> Make sure the IAM role is mapped to the instance for getting the STS get-caller-identity.

<script id="asciicast-378383" src="https://asciinema.org/a/378383.js" async></script>
