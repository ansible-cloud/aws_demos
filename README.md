# AWS Demonstration

## Table of Contents

- [Objective](#Objective)
- [AWS Resources](#AWS_Resources)
- [AWS Instance Provisioning](#AWS_Instance_Provisioning)
- [AWS Dynamic Inventory](#AWS_Dynamic_Inventory)

# Objective

Demonstrate using Ansible Playbooks to provision a RHEL 7.4 instance on Amazon Web Services (AWS).

This demonstration is broken into two parts, configuring AWS resources (such as a VPC) and then spinning up an AWS instance itself, then using dynamic inventory to connect to the instance.

The following modules will be demonstrated in the [provision_resources.yml](provision_resources.yml) Playbook:
- [ec2_key](https://docs.ansible.com/ansible/latest/modules/ec2_key_module.html) - create or delete an ec2 key pair
- [ec2_vpc_net](https://docs.ansible.com/ansible/latest/modules/ec2_vpc_net_module.html) - Configure AWS virtual private clouds
- [ec2_group](https://docs.ansible.com/ansible/latest/modules/ec2_group_module.html) - maintain an ec2 VPC security group.
- [ec2_vpc_subnet](https://docs.ansible.com/ansible/latest/modules/ec2_vpc_subnet_module.html) - Manage subnets in AWS virtual private clouds
- [ec2_vpc_igw](https://docs.ansible.com/ansible/latest/modules/ec2_vpc_igw_module.html) - Manage an AWS VPC Internet gateway
- [ec2_vpc_route_table](https://docs.ansible.com/ansible/latest/modules/ec2_vpc_route_table_module.html) - Manage route tables for AWS virtual private clouds

The following modules will be demonstrated in the [provision_rhel.yml](provision_rhel.yml) Playbook:
- [ec2_ami_facts](https://docs.ansible.com/ansible/latest/modules/ec2_ami_facts_module.html) - Gather facts about ec2 AMIs
- [ec2](https://docs.ansible.com/ansible/devel/modules/ec2_module.html) - create, terminate, start or stop an instance in ec2

# Requirements

This demo assumes you have an AWS account and have setup your credentials already.  The Github Project [Linklight has detailed directions here](https://github.com/network-automation/linklight/blob/master/docs/setup.md) if you need help on setting up an AWS.

## AWS Resources

This Playbook does the following:
- Creates an SSH key pair to use for Instances
  - saves the private key to a file
- Creates an AWS Virtual Private Cloud (VPC)
- Creates an AWS Security Group (SG)
- Creates a VPC Subnet
  - saves this subnet-id to a file resources.yml
- Creates an Internet Gateway (IGW)
- Creates a Default Route to the Internet from the subnet through the IGW

Running the provision_resources.yml Playbook.

```
➜  aws_provision_instances ansible-playbook provision_resources.yml

PLAY [AWS PLAYBOOK 101] ********************************************************

TASK [Create ssh key pair] *****************************************************
changed: [localhost]

TASK [save private key] ********************************************************
changed: [localhost]

TASK [Create AWS VPC AWS-REINVENT-vpc] *****************************************
changed: [localhost]

TASK [Create EC2 security group AWS-REINVENT-SG] *******************************
changed: [localhost]

TASK [Create subnet for AWS-REINVENT-vpc] **************************************
changed: [localhost]

TASK [Save vpc_subnet_id to file] **********************************************
changed: [localhost]

TASK [create vpc internet gateway for AWS-REINVENT-vpc] ************************
changed: [localhost]

TASK [This creates a default route (0.0.0.0/0) for the subnet to] **************
changed: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=8    changed=8    unreachable=0    failed=0
```

## AWS Instance Provisioning

This Playbook does the following:
- Finds the correct AMI identifier for RHEL7 regardless of region
- provisions the RHEL 7.4 instance into the VPC
- grabs Dynamic Inventory from AWS (which will grab the newly provisioned instance)
- uses the `wait_for_connection` module to connect to the RHEL7 instance (which could be used for configuration management in the future).

Running the provision_rhel.yml Playbook.

```
➜  aws_provision_instances ansible-playbook provision_rhel.yml

PLAY [AWS PLAYBOOK 101] ********************************************************

TASK [Include vars of resources.yml which contains the vpc-subnet-id that we need] *
ok: [localhost]

TASK [find ami instance-id for RHEL7] ******************************************
ok: [localhost]

TASK [SET AMI FOR SERVER01] ****************************************************
ok: [localhost]

TASK [Create EC2 instances for server01 node] **********************************
changed: [localhost]

TASK [debug] *******************************************************************
ok: [localhost] => (item=ec2-35-167-240-253.us-west-2.compute.amazonaws.com) => {
    "item": "ec2-35-167-240-253.us-west-2.compute.amazonaws.com"
}

PLAY [use wait for connection] ********************************************************************************

TASK [wait to connect] *********************************************************
ok: [ec2-35-167-240-253.us-west-2.compute.amazonaws.com]

PLAY RECAP *********************************************************************
ec2-35-167-240-253.us-west-2.compute.amazonaws.com : ok=1    changed=0    unreachable=0    failed=0
localhost                  : ok=5    changed=0    unreachable=0    failed=0
```

## AWS Dynamic Inventory

This demonstration is making use of Ansible Dynamic Inventory through the use of a [Inventory Plugin](https://docs.ansible.com/ansible/latest/plugins/inventory.html).  There is a yaml file named [demo.aws_ec2.yml](demo.aws_ec2.yml) that is using the [aws_ec2 inventory plugin](https://docs.ansible.com/ansible/latest/plugins/inventory/aws_ec2.html) that is configured as shown here:

```
---
plugin: aws_ec2
regions:
  - us-west-2
filters:
  tag:Name: TEST_INSTANCE
```

This grabs instance information (such as IP address) dynamically from AWS so we can run additional tasks on the instance.  On the command line we can call this plugin with the `-i` command e.g. `ansible-playbook provision_rhel.yml -i demo.aws_ec2.yml` but in the case of this demonstration we set the inventory in the `ansible.cfg` file like this:
```
inventory      = demo.aws_ec2.yml
```

### Dynamic Inventory Strategy

In the Playbook we load the dynamic inventory by default.  By default there may be no instances actually provisioned so we only get the implicit localhost.  We need to reload the inventory after the `ec2` module provisions the instance.  To do this we use the `meta` module as shown here:

```
- name: grab new aws facts
  meta: refresh_inventory
```

### Author

[Sean Cavanaugh](https://twitter.com/ipvsean)

https://www.ansible.com/blog/author/sean-cavanaugh
