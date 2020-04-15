# Ansible role: AWS VPC Route tables

This role handles the creation of Route tables for a VPC in AWS.

[![Build Status](https://travis-ci.org/Flaconi/ansible-role-aws-vpc-route-table.svg?branch=master)](https://travis-ci.org/Flaconi/ansible-role-aws-vpc-route-table)
[![Version](https://img.shields.io/github/tag/Flaconi/ansible-role-aws-vpc-route-table.svg)](https://github.com/Flaconi/ansible-role-aws-vpc-route-table/tags)
[![Ansible Galaxy](https://img.shields.io/ansible/role/d/25922.svg)](https://galaxy.ansible.com/Flaconi/aws-vpc-route-table/)

## Requirements

* Ansible 2.5


## Additional variables

Additional variables that can be used (either as `host_vars`/`group_vars` or via command line args):

| Variable                                         | Description                  |
|--------------------------------------------------|------------------------------|
| `aws_vpc_route_table_profile`                    | Boto profile name to be used |
| `aws_vpc_route_table_default_region`             | Default region to use        |
| `aws_vpc_route_table_vpc_filter_additional`      | Additional `key` `val` filter to add to `vpc_filter` and `vpc_name` by default. |
| `aws_vpc_route_table_instance_filter_additional` | Additional `key` `val` filter to add to `instance_filter` and `instance_name` by default. |
| `aws_vpc_route_table_eni_filter_additional`      | Additional `key` `val` filter to add to `eni_filter` and `eni_name` by default. |
| `aws_vpc_route_table_nat_filter_additional`      | Additional `key` `val` filter to add to `nat_filter` and `nat_name` by default. |
| `aws_vpc_route_table_vgw_filter_additional`      | Additional `key` `val` filter to add to `vgw_filter` and `vgw_name` by default. |
| `aws_vpc_route_table_subnet_filter_additional`   | Additional `key` `val` filter to add to `subnet_filter` and `subnet_name` by default. |
| `aws_vpc_route_table_peering_filter_additional`  | Additional `key` `val` filter to add to `peering_filter` and `peering_name` by default. |
| `aws_vpc_route_table_purge_routes`               | Boolean flag for toggling the [union][1] of routes as described locally (Ansible) vs. remote (AWS). `Default: True` |
| `aws_vpc_route_table_purge_subnets`              | Boolean flag for toggling the [union][1] of subnets as described locally (Ansible) vs. remote (AWS). `Default: True` |
| `aws_vpc_route_table_purge_tags`                 | Boolean flag for toggling the [union][1] of route table tags as described locally (Ansible) vs. remote (AWS). `Default: True` |

__Important:__ the above three variables responsible for purging entries on AWS
are scoped for the whole role invocation, even though the role supports taking
in a list of route tables to handle.

## Example definition

#### Required parameter only

```yml
aws_vpc_route_tables:

  # Create route tables for a VPC by VPC name
  - vpc_name: devops-test-vpc
    route_tables:
      - name: my-route-table-1
        routes:
          - dest: 0.0.0.0/0
            type: igw
        subnets:
          - name: my-subnet-name-1

  # Create route tables for a VPC by VPC filter
  - vpc_filter:
      - key: "tag:Name"
        val: "devops-test-vpc"
      - key: "tag:env"
        val: playground
      - key: "tag:department"
        val: devops
    route_tables:
      - name: my-route-table-2
        routes:
          # Example for Internet gateway
          - dest: 0.0.0.0/0
            type: igw
          # Example for NAT GW by name
          - dest: 185.28.100.13/32
            type: nat
            name: my-nat-gw-name
          # Example for NAT GW by filter
          - dest: 185.28.100.13/32
            type: nat
            filter:
              - key: "tag:Name"
                val: my-nat-gw-name
              - key: "tag:env"
                val: production
          # Example for Instance by name
          - dest: 185.28.100.13/32
            type: instance
            name: my-instance-name
          # Example for Instance by filter
          - dest: 185.28.100.13/32
            type: instance
            filter:
              - key: "tag:Name"
                val: my-instance-name
              - key: "tag:env"
                val: production
          # Example for ENI by name
          - dest: 185.28.100.13/32
            type: eni
            name: my-eni-name
          # Example for ENI by filter
          - dest: 185.28.100.13/32
            type: eni
            filter:
              - key: "tag:Name"
                val: my-eni-name
              - key: "tag:env"
                val: production
          # Example for VPC Peering by name
          - dest: 185.28.100.13/32
            type: peering
            name: my-peering-name
          # Example for VPC Peering by filter
          - dest: 185.28.100.13/32
            type: peering
            filter:
              - key: "tag:Name"
                val: my-peering-name
              - key: "tag:env"
                val: production
          # Example for VGW by filter without route propagation
          - dest: 185.28.100.14/32
            type: vgw
            propagation: False
            name: my-vgw-name-1
          # Example for VGW by filter with route propagation
          - dest: 185.28.100.14/32
            type: vgw
            propagation: True
            filter:
              - key: "tag:Name"
                val: my-vgw-name
              - key: "tag:env"
                val: production
        subnets:
          # Example for subnet by name
          - name: my-subnet-name
          # Example for subnet by cidr
          - cidr: 172.25.2.0/24
          # Example for subnet by id
          - id: subnet-0fead839
          # Example for subnet by filter
          - filter:
              - key: "tag:Name"
                val: my-subnet-name
              - key: "tag:env"
                val: production
```

#### All available parameter
```yml
# Only gather available resources
aws_vpc_route_table_vpc_filter_additional:
  - key: state
    val: available

aws_vpc_route_table_ec2_filter_additional:
  - key: state
    val: available

aws_vpc_route_table_eni_filter_additional:
  - key: state
    val: available

aws_vpc_route_table_nat_filter_additional:
  - key: state
    val: available

aws_vpc_route_table_vgw_filter_additional:
  - key: state
    val: available

aws_vpc_route_table_subnet_filter_additional:
  - key: state
    val: available

aws_vpc_route_table_peering_filter_additional:
  - key: status-code
    val: active

# The next three vars help us ensure that existing routes, subnets associations
# and route table tags on AWS shall be kept. The new entries, as defined in
# Ansible, will be appended.
aws_vpc_route_table_purge_routes: False
aws_vpc_route_table_purge_subnets: False
aws_vpc_route_table_purge_tags: False

aws_vpc_route_tables:

  # Create route tables for a VPC by VPC name
  - vpc_name: devops-test-vpc
    region: eu-central-1
    route_tables:
      - name: my-route-table-1
        tags:
          - key: env
            val: production
          - key: department
            val: devops
        routes:
          - dest: 0.0.0.0/0
            type: igw
        subnets:
          - name: my-subnet-name-1

  # Create route tables for a VPC by VPC filter
  - vpc_filter:
      - key: "tag:Name"
        val: "devops-test-vpc"
      - key: "tag:env"
        val: playground
      - key: "tag:department"
        val: devops
    region: eu-central-1
    route_tables:
      - name: my-route-table-2
        tags:
          - key: env
            val: production
          - key: department
            val: devops
        routes:
          # Example for Internet gateway
          - dest: 0.0.0.0/0
            type: igw
          # Example for NAT GW by name
          - dest: 185.28.100.13/32
            type: nat
            name: my-nat-gw-name
          # Example for NAT GW by filter
          - dest: 185.28.100.13/32
            type: nat
            filter:
              - key: "tag:Name"
                val: my-nat-gw-name
              - key: "tag:env"
                val: production
          # Example for Instance by name
          - dest: 185.28.100.13/32
            type: instance
            name: my-instance-name
          # Example for Instance by filter
          - dest: 185.28.100.13/32
            type: instance
            filter:
              - key: "tag:Name"
                val: my-instance-name
              - key: "tag:env"
                val: production
          # Example for ENI by name
          - dest: 185.28.100.13/32
            type: eni
            name: my-eni-name
          # Example for ENI by filter
          - dest: 185.28.100.13/32
            type: eni
            filter:
              - key: "tag:Name"
                val: my-eni-name
              - key: "tag:env"
                val: production
          # Example for VPC Peering by name
          - dest: 185.28.100.13/32
            type: peering
            name: my-peering-name
          # Example for VPC Peering by filter
          - dest: 185.28.100.13/32
            type: peering
            filter:
              - key: "tag:Name"
                val: my-peering-name
              - key: "tag:env"
                val: production
          # Example for VGW by filter without route propagation
          - dest: 185.28.100.14/32
            type: vgw
            propagation: False
            name: my-vgw-name-1
          # Example for VGW by filter with route propagation
          - dest: 185.28.100.14/32
            type: vgw
            propagation: True
            filter:
              - key: "tag:Name"
                val: my-vgw-name
              - key: "tag:env"
                val: production
        subnets:
          # Example for subnet by name
          - name: my-subnet-name
          # Example for subnet by cidr
          - cidr: 172.25.2.0/24
          # Example for subnet by id
          - id: subnet-0fead839
          # Example for subnet by filter
          - filter:
              - key: "tag:Name"
                val: my-subnet-name
              - key: "tag:env"
                val: production
```


## Testing

#### Requirements

* Docker
* [yamllint](https://github.com/adrienverge/yamllint)

#### Run tests

```bash
# Lint the source files
make lint

# Run integration tests with default Ansible version
make test

# Run integration tests with custom Ansible version
make test ANSIBLE_VERSION=2.4
```

[1]: https://en.wikipedia.org/wiki/Union_(set_theory)
