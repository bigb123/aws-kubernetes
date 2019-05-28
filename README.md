# aws-kubernetes
Cloudofrmation script to deploy skeleton of AWS EKS. It will be deployed in
us-east-1 region in availability zone A and B.

Among many resources that will be created by the template the most interesting
of them are:
- VPC
- two public subnets
- two private subnets with nat gateways
- ECS Cluster
- two EC2 t2.medium instances in private subnets that will act as nodes

This stack requires to keep Github token and database secrets in AWS Secrets
Manager. Things to do:
- Create new secret for Github token with one key. You can name the key "Token".
To get token from Github go to https://github.com/settings/tokens and generate
one.
- Provide secret's name and key to the template's parameters before runtime.

# Manual deployment
If you're deploying this stack "by hand" comment out "EKSAccessUser" section in
"eks-template.yaml" file.

# Automatic deployment

This template is designed to run with
https://github.com/bigb123/aws-ci-cd-pipeline-github. aws-ci-cd-pipeline-github
builds AWS Code Pipeline to automatically deploy Kubernetes template on Cloudformation
when new commit that modifies "eks-wordpress.yaml" file will arrive in
this repository.

## Kubernetes initial configuration
Use this instruction only if you are deploying this stack non-manually.

To initially configure Kubernetes cluster you have to use the same role that
was used to create cluster. This is the only way to access the cluster.

This stack creates such user. To find the user in IAM check "Outputs" section of the
stack and use "EKSAccessUserName" value.

Next thing you have to do is to make sure you have CLI access to your AWS
account using this user. You can obtain CLI credentials using IAM web console.

Next, edit $HOME/.aws/config by adding profile that will be used with kubectl
commands. It can looks like the one below:

```
[default]
region = us-east-1
output = json

[profile eksaccess]
region = us-east-1
output = json
role_arn = arn:aws:iam::<account_number>:role/Actual_Pipeline_Cloudformation_deployment_role
source_profile = default
```

Details on https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-cli.html?shortFooter=true

Also, add `sts:assumeRole` to the deployment role's trust relationship just after
"PipelineRole" record:
```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::<account number>:user/<username>"
  },
  "Action": "sts:AssumeRole"
}
```
The user ARN is available in this stack's Output section.

Then, make sure the `kubectl` and `aws-iam-authenticator` are available in your
terminal. You can follow this link
https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html section
"Step 2: Create a kubeconfig File"
