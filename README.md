# IAM EKS User Mapper


# What it does
If you are using EKS for your cluster and IAM to manage your AWS users, this tool allows you to create and manage EKS users and their groups on the basis of your IAM groups with fine-grained access control.

More about IAM Groups: [https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups.html](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups.html)
More about Kubernetes RBAC: [https://kubernetes.io/docs/reference/access-authn-authz/rbac/](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

### Example usecase
Assuming you have 2 IAM groups `devops` and `devs`, and you want to give devops's members `system:masters` role and devs a `developer` role, you can do that via:

`./app --iam-k8s-group=devops::system:masters,devs::developer`

See the [usage](#Usage) section below for help.

## Usage

Provide comma separated values for iam-k8s mapping with each mapping represented as
<iam>::<k8s-group>.

Example usages
```bash

# To map all your devops IAM Group as system:masters and devs IAM Group as developer
./app --iam-k8s-group=devops::system:masters,devs::developer

# Support for multiple kubernetes roles (use `|` between K8s roles)
# To map all your devops IAM Group as system:masters and devs IAM Group as both developer and manager
./app --iam-k8s-group=devops::system:masters,devs::developer|manager
```

## Setting up
1. Have an AWS IAM Group with users that you want to have access to your EKS cluster
2. Create a new IAM User with an IAM ReadOnly policy (or) a new IAM role with IAM ReadOnly Policy and capability to assume role.
3.
  - If you are using an IAM Role, add the ARN for it in `kubernetes/deployment.yml` path: `spec.template.metadata.annotations` with annotation `iam.amazonaws.com/role`: `ROLE_ARN`
  - If you are using an IAM User, add environment variables `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` in `kubernetes/deployment.yml` path: `spec.template.spec.containers.0.env`
4. If you manage multiple AWS Accounts and want to fetch IAM roles from a different account than your current cluster's, set the env var `AWS_IAM_ACCOUNT_ROLE_ARN` in your deployment.yml, else remove the variable from the deployment spec.
5. Edit the `kubernetes/deployment.yml` `command:` with both the IAM group name you want to provide access to, and the Kubernetes group each user in the group should be mapped to.
6. Finally:
```bash
$ kubectl apply -f kubernetes/
```

## How it works
EKS uses a configmap `aws-auth` in the namespace `kube-system`. It manages your cluster access via aws-iam-authenticator using that configmap. This tool queries the IAM groups and updates the configmap with the given kubernetes roles

## Planned features

1. Option to give specific IAM Users access as well (will be a union if that user is part of a provided IAM group as well)
2. Improved CLI Experience

## Thanks

Inspired from [ygrene's](https://github.com/ygrene) [IAM EKS Mapper](https://github.com/ygrene/iam-eks-user-mapper)
