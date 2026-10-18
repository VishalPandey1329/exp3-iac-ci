# Experiment III - IaC CI Pipeline

This repository runs Terraform formatting, initialization and validation, TFLint, tfsec, and a Terraform plan in Jenkins on Windows.

## Pipeline behavior

- The default `APPLY_INFRASTRUCTURE=false` run makes a plan with refresh disabled. This lets CI validate the Terraform graph without AWS credentials or AWS API calls. It is a CI-only plan and must not be used as an apply plan.
- To request a refreshed AWS plan, start a build with `APPLY_INFRASTRUCTURE=true`. Jenkins must have AWS credentials stored in its credential store and exposed to the build. Never commit AWS credentials or `.tfstate` files.
- Applying is limited to `main` and still requires both Jenkins approval prompts. Do not approve an apply unless you intend to create or change AWS resources.

## Security settings

The sample restricts HTTP and outbound security group traffic to the VPC CIDR, disables automatic public IP assignment, requires IMDSv2 tokens, and encrypts the EC2 root volume.
