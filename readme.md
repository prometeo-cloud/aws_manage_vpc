# aws_manage_vpc

## Description:

Manages AWS Virtual Private Clouds (VPCs) as follows:

- Running the role will create the VPCs defined in `aws_vpcs` when the variable `arg_action` is not defined or is defined and set to `create`
- Running the role will delete the VPCs defined in `aws_vpcs` and any associated elastic IP addresses and Route Table routes when the variable `arg_action` is defined and is set to `delete` (note: ensure that the VPCs have no resources or they will not be deleted)
- Running the [get_vpc_facts.yml](./tasks/get_vpc_facts.yml) play will return information about the specified VPC.

This role requires Boto3 to be installed as follows:

`sudo pip install boto3`

or:

```yaml
- name: Install boto3
  easy_install:
    name: boto3
    state: present
```

## Dependencies

This role uses the AWS cli.  It therefore requires AWS credentials as environment variables or in a configuration file (see [here](https://docs.aws.amazon.com/cli/latest/topic/config-vars.html#cli-aws-help-config-vars)).  The role `aws_credentials` will create a config file for use with the AWS cli.

## Behaviour:

**Feature:** Create AWS VPCs  
- **Given** valid AWS credentials
- **When** the script is executed
- **Then** the VPCs are created if they do not exist

## Configuration:

Common variables used by this role:

| Variable | Description | Example |
|-----|-----|-----|
| **aws_region** | AWS region | See [AWS regions](http://docs.aws.amazon.com/general/latest/gr/rande.html#ec2_region) |
| **aws_access_key** | AWS access key | AKIAIOSFODNN7EXAMPLE |
| **aws_secret_key** | AWS secret key | wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY |
| **resource_tags** | List of resource tags | see usage |

Variables specific to this role:

| Variable | Description | Example |
|-----|-----|-----|
| **aws_vpcs** | List of VPCs to create. | see usage |
| **vpc_tag** | The `Name:` tag of the VPC to get information about. | see usage |

## Usage:

How to invoke the role from a playbook (create VPCs):

```yaml
- name: Create AWS VPCs
  include_role:
    name: "aws_manage_vpc"
  vars:
    aws_access_key: AKIAIOSFODNN7EXAMPLE
    aws_secret_key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
    aws_region: eu-west-2  # London
    resource_tags:
      Name: "{{ resource_name }}"   # Set to upper case aws_vpc_name
      Owner: "Acme Co."

    aws_vpcs:
      - aws_vpc_name: "my-test-vpc-a"
        aws_vpc_cidr_main:
          - "10.11.0.0/16"
        aws_vpc_cidrs_additional:
          - "10.22.0.0/16"
          - "10.33.0.0/16"
        aws_vpc_region: "{{ aws_region }}"
        aws_vpc_state: "present"
        aws_vpc_tenancy: "default"
        aws_vpc_tags: "{{ resource_tags }}"
    
      - aws_vpc_name: "my-test-vpc-b"
        aws_vpc_cidr_main:
          - "10.44.0.0/16"
        aws_vpc_cidrs_additional:
          - "10.55.0.0/16"
          - "10.66.0.0/16"
        aws_vpc_region: "{{ aws_region }}"
        aws_vpc_state: "present"
        aws_vpc_tenancy: "default"
        aws_vpc_tags: "{{ resource_tags }}"
```

How to invoke the role from a playbook (delete VPCs):

```yaml
- name: Delete AWS VPCs
  include_role:
    name: "aws_manage_vpc"
  vars:
    arg_action: "delete"

    aws_access_key: AKIAIOSFODNN7EXAMPLE
    aws_secret_key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
    aws_region: eu-west-2  # London
    resource_tags:
      Name: "{{ resource_name }}"   # Set to upper case aws_vpc_name
      Owner: "Acme Co."

    aws_vpcs:
      - aws_vpc_name: "my-test-vpc-a"
        aws_vpc_cidrs:
          - "10.11.0.0/16"
        aws_vpc_region: "{{ aws_region }}"
    
      - aws_vpc_name: "my-test-vpc-b"
        aws_vpc_cidrs:
          - "10.44.0.0/16"
        aws_vpc_region: "{{ aws_region }}"
```

How to get information about a VPC:

```yaml
- name: Get VPC Information
  include_role:
    name: "aws_manage_vpc"
    tasks_from: "get_vpc_facts"
  vars:
    vpc_tag: "Test A"
```

This will return facts in the variable `vpc_net_facts`.  The data returned is as follows:

```yaml
"vpc_net_facts": {
    "changed": false, 
    "failed": false, 
    "vpcs": [
        {
            "cidr_block": "10.10.0.0/22", 
            "classic_link_enabled": null, 
            "dhcp_options_id": "dopt-1c4d5725", 
            "id": "vpc-b59de2af", 
            "instance_tenancy": "default", 
            "is_default": false, 
            "state": "available", 
            "tags": {
                "Name": "Test A"
            }
        }
    ]
}
```
