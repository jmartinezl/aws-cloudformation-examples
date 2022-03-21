# AWS CloudFormation Sample Templates
Use sample AWS CloudFormation templates to learn how to declare specific AWS resources or solve a particular use case. We recommend that you use sample templates as a starting point for creating your own templates, not for launching production-level environments. Before launching a template, always review the resources that it will create and the permissions it requires.

## CLI use guide

You use the CLI to fire up a Cloudformation stack using the create-stack command. The command, however, takes a few arguments to pass important information. This minimal example shows you how to point CloudFormation to your JSON template file, a name to assign to your stack, and a valid SSH key so I’ll be able to log into the instance it creates.

```bash
aws cloudformation create-stack \
  --template-body file://file.yaml \
  --stack-name lamp \
  --parameters \
  ParameterKey=KeyName,ParameterValue=mykey
```
