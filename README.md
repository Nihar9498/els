# ansible-aws-elasticsearch


### Requirements:

- Ansible
- boto

### AWS credentials:

Ansible uses python-boto library to call AWS API, and boto needs AWS credentials in order to perform all the functions. There are many ways to configure your AWS credentials. The easiest way is to crate a .boto file under your user home directory:
```shell
ssh-keygen -t rsa -b 4096 -f ~/.ssh/my_aws
ansible-vault create group_vars/all/pass.yml
New Vault password:
Confirm New Vault password:
```

Then add the following:
```ansible-vault edit group_vars/all/pass.yml 
Vault password:
ec2_access_key: AAAAAAAAAAAAAABBBBBBBBBBBB                                      
ec2_secret_key: afjdfadgf$fgajk5ragesfjgjsfdbtirhf
```

### To use this Role:

Edit the `group_vars/all.yml` vars file:
```yaml
cluster: elasticsearch

# Tags
# if you'll change `Name` then change it in `setup_elasticsearch.yml` twice too ;)
tags:
  Name: "{{ cluster }}"

# ec2_key
key_name: "my_aws"
ssh_key_location: ~/.ssh/id_rsa.pub 

# VPC
vpc_region: us-east-1
vpc_name: "{{ cluster }}-vpc"
vpc_cidr_block: 172.22.0.0/16
vpc_subnets:
  - cidr: 172.22.1.0/24
    az: "{{ vpc_region }}a"
    resource_tags: { "Name":"{{ cluster }}" }
  - cidr: 172.22.2.0/24
    az: "{{ vpc_region }}c"
    resource_tags: { "Name":"{{ cluster }}" }
sg_name: "{{ cluster }}-security-group"

# EC2
image: ami-ff427095 # ubuntu trusty 14.04
instances_count: 1 # count per one subnet
instances_type: t2.micro
instances_data_size: 10
instances_device_name: /dev/sdb

# LB
lb_name: "{{ cluster }}-load-balancer"

# ElasticSearch
es_mount_point: /data
es_cluster_name: "{{ cluster }}"
es_data_dir: "{{ es_mount_point }}/elasticsearch/data"
```

### Create aws environment

```shell
ansible-playbook setup_aws.yml
```

What will be created:
- Ssh key pair
- Network security group that allows any ssh traffic and internal vpc traffic
- IAM role with policy that allows ec2 discovery
- Ec2 instances in all vpc networks
- Internal load balancer

### Install Elasticsearch

```shell
ansible-playbook -i inventory/ setup_elasticsearch.yml
```

Warning: repeated application **will restart** worked nodes in most cases. Use `limit` if you want apply it only to new hosts!

What will be created:
- Mount /dev/sdb as /data on instances
- Install Elasticsearch 2.x
- Install cloud-aws plugin for elasticsearch
- Set java heap size as half or ram for elasticsearch
- Configure autodiscovery via ec2 tags for elasticsearch nodes
- Configure SSL
