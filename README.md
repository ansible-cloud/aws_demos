## AWS Demos - Playbooks Overview

This repository contains Ansible playbooks and roles for demonstrating AWS automation and reporting. Below is a quick reference to the playbooks under the `playbooks` directory. The `collections` directory is intentionally ignored here.

### Table of Contents
- [AWS Demos - Playbooks Overview](#aws-demos---playbooks-overview)
  - [Table of Contents](#table-of-contents)
  - [Prerequisites](#prerequisites)
  - [How to run](#how-to-run)
  - [Playbooks](#playbooks)
    - [playbooks/aws\_resources.yml](#playbooksaws_resourcesyml)
    - [playbooks/aws\_instances.yml](#playbooksaws_instancesyml)
    - [playbooks/aws\_ssm.yml](#playbooksaws_ssmyml)
    - [playbooks/aws\_ssm\_assume.yml](#playbooksaws_ssm_assumeyml)
    - [playbooks/cloud\_report.yml](#playbookscloud_reportyml)
    - [playbooks/cloud\_report\_tags.yml](#playbookscloud_report_tagsyml)
    - [playbooks/info\_combined.yml](#playbooksinfo_combinedyml)
    - [playbooks/info\_vpcs.yml](#playbooksinfo_vpcsyml)
    - [playbooks/info\_instances.yml](#playbooksinfo_instancesyml)
    - [playbooks/info\_igws.yml](#playbooksinfo_igwsyml)
    - [playbooks/lab2-deploy-application.yml](#playbookslab2-deploy-applicationyml)
    - [playbooks/lab2-ec2-info.yml](#playbookslab2-ec2-infoyml)
    - [playbooks/lab2-patching.yml](#playbookslab2-patchingyml)
    - [playbooks/lab3-challenge2.yml](#playbookslab3-challenge2yml)
    - [playbooks/tag\_info\_aws.yml](#playbookstag_info_awsyml)
    - [playbooks/stop\_aws\_ec2\_instances.yml](#playbooksstop_aws_ec2_instancesyml)
  - [Notes](#notes)
  - [License](#license)
### Prerequisites
- Ansible 2.12+ and required AWS collections (`amazon.aws`, `community.aws`, `awx.awx`).
- AWS credentials configured (env vars, shared credentials file, or instance profile).
- Default region is `us-east-1` unless overridden.

### How to run
Use `-e` to override variables when needed.

```bash
ansible-playbook playbooks/<playbook>.yml -e "key=value ..."
```

### Playbooks

#### playbooks/aws_resources.yml
Creates foundational AWS resources for demos:
- SSH key pair saved locally as `ansible-demo.pem` (and optional AWX/Tower machine credential)
- VPC, subnet, Internet Gateway, route table
- Security group for SSH and HTTP

Example:
```bash
ansible-playbook playbooks/aws_resources.yml -e "ec2_region=us-east-1 ec2_name_prefix=ansible-demo ec2_cidr=192.168.0.0/28"
```

---

#### playbooks/aws_instances.yml
Launches tagged RHEL EC2 instances in the created VPC/subnet and applies tags `ansible-demo=true`, `instruqt=true`, and indexed names.

Variables (examples): `ec2_region`, `ec2_vpc_subnet_name`, `ec2_key_name`, `ec2_security_group`, `ec2_instance_type`, `ec2_exact_count`.

Example:
```bash
ansible-playbook playbooks/aws_instances.yml -e "ec2_region=us-east-1 ec2_vpc_subnet_name=ansible-demo ec2_exact_count=2"
```

---

#### playbooks/aws_ssm.yml
Runs commands on instances via AWS Systems Manager (SSM) connection plugin and creates a test file.

Variables: `ansible_aws_ssm_region`, `ansible_aws_ssm_bucket_name`, `ansible_aws_ssm_instance_id`, `ansible_user`.

Example:
```bash
ansible-playbook playbooks/aws_ssm.yml -i "ssm," -e "ansible_aws_ssm_region=us-east-1 ansible_aws_ssm_instance_id=i-xxxxxxxxxxxxxx ansible_user=ssm-user"
```

---

#### playbooks/aws_ssm_assume.yml
Assumes an IAM role for SSM/S3 access, runs a file task over SSM, and demonstrates limited permissions.

Example:
```bash
ansible-playbook playbooks/aws_ssm_assume.yml -i "ssm,"
```

---

#### playbooks/cloud_report.yml
Builds a multi-part cloud report using roles:
- Collect EC2 instance facts
- Aggregate general info
- Template an HTML report
- Publish the report either to a Linux host or to S3 (when running on localhost)

Example:
```bash
ansible-playbook playbooks/cloud_report.yml -e "_aws_instances=tag_Name_rhel* _hosts=localhost"
```

---

#### playbooks/cloud_report_tags.yml
Generates and publishes a tags-focused AWS report using the `build_report_tags` role.

Example:
```bash
ansible-playbook playbooks/cloud_report_tags.yml
```

---

#### playbooks/info_combined.yml
Retrieves and prints combined info for VPCs, EC2 instances, and Internet Gateways using a template for display.

Example:
```bash
ansible-playbook playbooks/info_combined.yml -e "ec2_region=us-east-1"
```

---

#### playbooks/info_vpcs.yml
Lists VPCs and prints raw details.

Example:
```bash
ansible-playbook playbooks/info_vpcs.yml -e "ec2_region=us-east-1"
```

---

#### playbooks/info_instances.yml
Lists EC2 instances and prints raw details.

Example:
```bash
ansible-playbook playbooks/info_instances.yml -e "ec2_region=us-east-1"
```

---

#### playbooks/info_igws.yml
Lists Internet Gateways and prints raw details.

Example:
```bash
ansible-playbook playbooks/info_igws.yml -e "ec2_region=us-east-1"
```

---

#### playbooks/lab2-deploy-application.yml
Installs a given Linux application (or list) via `dnf` on target hosts.

Variables: `HOSTS`, `application`.

Example:
```bash
ansible-playbook playbooks/lab2-deploy-application.yml -e "HOSTS=rhel1 application=git"
```

---

#### playbooks/lab2-ec2-info.yml
Displays selected EC2 instance fields in a formatted manner.

Variables: `your_region`, `your_tag` (filter value pattern).

Example:
```bash
ansible-playbook playbooks/lab2-ec2-info.yml -e "your_region=us-east-1 your_tag=rhel*"
```

---

#### playbooks/lab2-patching.yml
Runs Linux patching and generates reports using custom `demo.patching.*` roles; publishes a landing page.

Variables: `HOSTS`, optional `report_server` (defaults to `rhel1`).

Example:
```bash
ansible-playbook playbooks/lab2-patching.yml -e "HOSTS=rhel_group report_server=rhel1"
```

---

#### playbooks/lab3-challenge2.yml
Identifies EC2 instances matching a tag filter, prints counts and details, and exports their IDs to a workflow variable `identified_instances` for later steps.

Variables: `your_region`, `filter_input` (YAML string for `filters`).

Example:
```bash
ansible-playbook playbooks/lab3-challenge2.yml -e "your_region=us-east-1"
```

---

#### playbooks/tag_info_aws.yml
Prints VPC IDs/tags and EC2 instance tags for quick auditing.

Example:
```bash
ansible-playbook playbooks/tag_info_aws.yml -e "your_region=us-east-1"
```

---

#### playbooks/stop_aws_ec2_instances.yml
Stops EC2 instances whose IDs were previously captured into `identified_instances` (e.g., from `lab3-challenge2.yml`).

Variables: `your_region`, `identified_instances`.

Example:
```bash
ansible-playbook playbooks/stop_aws_ec2_instances.yml -e "your_region=us-east-1 identified_instances='["i-abc","i-def"]'"
```

---

### Notes
- Many playbooks default to `us-east-1`; override with `-e ec2_region=...` or `-e your_region=...`.
- Some reports publish to S3 when run against `localhost`; ensure proper permissions.
- For SSM-based playbooks, ensure the instance has SSM agent running and proper IAM role.

### License
This project is licensed under the GNU General Public License v3.0 or later (GPL-3.0-or-later). See the `LICENSE` file for details.
