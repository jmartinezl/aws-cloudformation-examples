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


# VPC Cloudformation example

## Storing parameters for your templates

The [AWS System Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-paramstore.html) are used to that end. These will allow you to set up configuration parameters
per account, per region to enable environment and network separation and keep
your resources organized.

When used with [Stack Sets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/what-is-cfnstacksets.html),
it allows you to pre-define configuration in your planned accounts and regions,
so that when the stack set is created, you can have different configurations for
different resources. This matters when it comes to CIDR Ranges, so that your
AWS VPC Networks do not overlap, which allows [VPC Peering](https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-peering.html).

Philosophically, the amount of System Manager Parameters created should be kept
to a minimum.

## Usage

#### Environment

Change these values based on your requirements.
At the very least, set the `VPNSGALLOWIP` parameter to the IP you desire to be able to access it from.

* NAME corresponds to the name of the Cloudformation Stack.
* ENVIRONMENT can be whatever you desire, and will be used by other "child
  stacks" to connect resources together. For example, the subnets created by this
  stack are added to the AWS System Manager Parameter Store, and ENVIRONMENT
  is the identifier that allows the "child stacks" to look up the values of
  subnets when needed for creating resources like Auto Scaling Groups.
  It can be whatever you like, but generally is one of:
  * management
  * shared-services
  * development
  * integration
  * qa
  * testing
  * staging
  * production
  * your-env-name-here

* CIDRBLOCK is the first IP address in range of ~65K IP Addresses (/16) that will be assigned to your VPC. Do not include the /16, this is appended by the template.
* VPNSGALLOWIP is the IP Address (or range) that you will allow connection over port 22 and 8080 to the VPN instance. (Created from a separate template, link to come)
  * Your current IP: `curl icanhazip.com`
* HA means High Availability, which creates 3 NAT Gateways instead of 1. See the `Expected Costs` section below for more information.

```
export NAME=vpc
export ENVIRONMENT=dev
export CIDRBLOCK=10.10.0.0
export VPNSGALLOWIP=192.168.0.1/32
export HA=false
```

#### Create Parameters
```
aws ssm put-parameter --name "Environment" --type "String" --value "${ENVIRONMENT}" --overwrite
aws ssm put-parameter --name "CidrBlock" --type "String" --value "${CIDRBLOCK}" --overwrite
aws ssm put-parameter --name "VPNSGAllowIP" --type "String" --value "${VPNSGALLOWIP}" --overwrite
aws ssm put-parameter --name "HA" --type "String" --value "${HA}" --overwrite
```

#### VPC Creation
```
aws cloudformation create-stack \
--stack-name ${NAME} \
--capabilities CAPABILITY_IAM \
--template-body file://vpc.yml \
--parameters \
ParameterKey=Environment,ParameterValue=Environment \
ParameterKey=CidrBlock,ParameterValue=CidrBlock \
ParameterKey=VPNSGAllowIP,ParameterValue=VPNSGAllowIP \
ParameterKey=HA,ParameterValue=HA

```

Output will be the Cloudformation Resource ID created.

##### VPC Update
```
aws cloudformation update-stack \
--stack-name ${NAME} \
--capabilities CAPABILITY_IAM \
--template-body file://vpc.yml \
--parameters \
ParameterKey=Environment,ParameterValue=Environment \
ParameterKey=CidrBlock,ParameterValue=CidrBlock \
ParameterKey=VPNSGAllowIP,ParameterValue=VPNSGAllowIP \
ParameterKey=HA,ParameterValue=HA
```

Output will be the Cloudformation Resource ID updated.

##### VPC Deletion
```
aws cloudformation delete-stack \
--stack-name ${NAME}
```

No output expected.
