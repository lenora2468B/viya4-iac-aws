# List of valid configuration variables

Supported configuration variables are listed in the table below.  All variables can also be specified on the command line.  Values specified on the command line will override all values in configuration defaults files.

## Table of Contents

- [List of valid configuration variables](#list-of-valid-configuration-variables)
  - [Table of Contents](#table-of-contents)
  - [Required Variables](#required-variables)
    - [AWS Authentication](#aws-authentication)
      - [Using Static Credentials](#using-static-credentials)
      - [Using AWS Profile](#using-aws-profile)
  - [Admin Access](#admin-access)
  - [General](#general)
  - [Nodepools](#nodepools)
    - [Default Nodepool](#default-nodepool)
    - [Additional Nodepools](#additional-nodepools)
  - [Storage](#storage)
    - [NFS Server](#nfs-server)
    - [AWS Elastic File System (EFS)](#aws-elastic-file-system-efs)
  - [Postgres](#postgres)

Terraform input variables can be set in the following ways:

- Individually, with the [`-var` command line option](https://www.terraform.io/docs/configuration/variables.html#variables-on-the-command-line).

- In [variable definitions (.tfvars) files](https://www.terraform.io/docs/configuration/variables.html#variable-definitions-tfvars-files). We recommend this way for most variables.
- As [environment variables](https://www.terraform.io/docs/configuration/variables.html#environment-variables). We recommend this way for the variables that set the [AWS authentication](#aws-authentication).

## Required Variables

| <div style="width:50px">Name</div> | <div style="width:150px">Description</div> | <div style="width:50px">Type</div> | <div style="width:75px">Default</div> | <div style="width:150px">Notes</div> |
| :--- | :--- | :--- | :--- | :--- |
| prefix | A prefix used in the name of all the AWS resources created by this script. | string | | The prefix string must start with a lowercase letter and contain only alphanumeric characters and dashes (-), but cannot end with a dash. |
| location | The AWS Region to provision all resources in this script | string | "us-east-1" | |
| ssh_public_key | Name of file with public ssh key for VMs | string | "~/.ssh/id_rsa.pub" | Value is required in order to access your VMs |

### AWS Authentication

The Terraform process manages AWS resources on your behalf. In order to do so, it needs to know the credentials for an AWS identity with the required permissons.

You can use either static credentials, or the name of an AWS Profile. If both are specified, the static credentials will take precedence. For recommendation on how to set these variables in your environment, see [Authenticating Terraform to access AWS](./user/TerraformAWSAuthentication).

#### Using Static Credentials

| <div style="width:50px">Name</div> | <div style="width:150px">Description</div> | <div style="width:50px">Type</div> | <div style="width:75px">Default</div> | <div style="width:150px">Notes</div> |
| :--- | :--- | :--- | :--- | :--- |
| aws_access_key_id | static credential key | string | "" | |
| aws_secret_access_key | static credential secret | string | "" | |
| aws_session_token | session token for validating temporary AWS credentials | string | "" | required only when using temporary AWS credentials|

#### Using AWS Profile

| <div style="width:50px">Name</div> | <div style="width:150px">Description</div> | <div style="width:50px">Type</div> | <div style="width:75px">Default</div> | <div style="width:150px">Notes</div> |
| :--- | :--- | :--- | :--- | :--- |
| aws_profile | name of AWS Profile in the credentials file | string | "" | |
| aws_shared_credentials_file | path to credentials file | string | [`~/.aws/credentials` on Linux and macOS](https://docs.aws.amazon.com/credref/latest/refdocs/file-location.html) | can be ignored when using the default value |

## Admin Access

By default, the API of the AWS resources that are being created are only accessible through authenticated AWS clients (e.g. the AWS Portal, the `aws` CLI, etc.)
To allow access for other administrative client applications (for example `kubectl`, `psql`, etc.), you want to open up the AWS firewall to allow access from your source IPs.
To do this, specify ranges of IP in [CIDR notation](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing).
Contact your Network System Administrator to find the public CIDR range of your network.

You can use `default_public_access_cidrs` to set a default range for all created resources. To set different ranges for other resources, define the appropriate variable. Use and empty list `[]` to disallow access explicitly.

| <div style="width:50px">Name</div> | <div style="width:150px">Description</div> | <div style="width:50px">Type</div> | <div style="width:75px">Default</div> | <div style="width:150px">Notes</div> |
| :--- | :--- | :--- | :--- | :--- |
| default_public_access_cidrs | IP Ranges allowed to access all created cloud resources | list of strings | | Use to to set a default for all Resources |
| cluster_endpoint_public_access_cidrs | IP Ranges allowed to access the AKS cluster api | list of strings | | for client admin access to the cluster, e.g. with `kubectl` |
| vm_public_access_cidrs | IP Ranges allowed to access the VMs | list of strings | | opens port 22 for SSH access to the jump and/or nfs VM |
| postgres_access_cidrs | IP Ranges allowed to access the AWS PostgreSQL Server | list of strings |||

## General

| <div style="width:50px">Name</div> | <div style="width:150px">Description</div> | <div style="width:50px">Type</div> | <div style="width:75px">Default</div> | <div style="width:150px">Notes</div> |
| :--- | :--- | :--- | :--- | :--- |
| create_static_kubeconfig | Allows the user to create a provider / service account based kube config file | bool | false | A value of `false` will default to using the cloud providers mechanism for generating the kubeconfig file. A value of `true` will create a static kubeconfig which utilizes a `Service Account` and `Cluster Role Binding` to provide credentials. |
| kubernetes_version | The EKS cluster K8S version | string | "1.18" | |
| create_jump_vm | Create bastion host | bool | true| |
| create_jump_public_ip | Add public ip to jump VM | bool | true | |
| jump_vm_admin | OS Admin User for the Jump VM | string | "jumpuser" | |
| jump_rwx_filestore_path | File store mount point on Jump server | string | "/viya-share" | This location cannot include "/mnt" as it's root location. This disk is ephemeral on Ubuntu which is the operating system being used for the Jump/NFS servers. |
| tags | Map of common tags to be placed on all AWS resources created by this script | map | { project_name = "viya" } | |

## Nodepools

### Default Nodepool

| <div style="width:50px">Name</div> | <div style="width:150px">Description</div> | <div style="width:50px">Type</div> | <div style="width:75px">Default</div> | <div style="width:150px">Notes</div> |
| :--- | :--- | :--- | :--- | :--- |
| default_nodepool_vm_type | Type of the default nodepool VMs | string | "m5.2xlarge" | |
| default_nodepool_os_disk_type | Disk type for default nodepool VMs | string | gp2 | |
| default_nodepool_os_disk_size | Disk size for default nodepool VMs in GB | number | 200 ||
| default_nodepool_os_disk_iops | Disk iops for default nodepool VMs | number | | For `io1` you MUST set to your desired IOPS value. Reference [Amazone EBS volume types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html) for details on values based on the `default_nodepool_os_disk_type` selected.|
| default_nodepool_node_count | Number of initial nodes in the default nodepool | number | 1 | The value must be between `default_nodepool_min_nodes` and `default_nodepool_max_nodes`|
| default_nodepool_max_nodes | Maximum number of nodes for the default nodepool | number | 5 | |
| default_nodepool_min_nodes | Minimum and initial number of nodes for the nodepool | number | 1 | |
| default_nodepool_taints | Taints for the default nodepool VMs | list of strings | | |
| default_nodepool_labels | Labels to add to the dfeault nodepool VMs | map | | |
| default_nodepool_custom_data | Additional userdata that will be appended to the default userdata. | string | "" | The value must be an empty string "" or the path to a file containing a `bash` script snippet that will be executed on the node pool. |

### Additional Nodepools

Additional node pools can be created separate from the default nodepool. This is done with the `node_pools` variable which is a map of objects. Each nodepool requires the following variables:

| <div style="width:50px">Name</div> | <div style="width:150px">Description</div> | <div style="width:50px">Type</div> | <div style="width:75px">Default</div> | <div style="width:150px">Notes</div> |
| :--- | :--- | :--- | :--- | :--- |
| vm_type | Type of the nodepool VMs | string | | |
| os_disk_type | Disk type for nodepool VMs | string | | `gp2` or `io1` |
| os_disk_size | Disk size for nodepool VMs in GB | number | | |
| os_disk_iops | Disk size for nodepool VMs in GB | number | | For `io1` you MUST set to your desired IOPS value. Reference [Amazone EBS volume types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html) for details on values based on the `os_disk_type` selected.|
| min_nodes | Minimum number of nodes for the nodepool | number | | The value must be between `min_nodes` and `max_nodes`|
| max_nodes | Maximum number of nodes for the nodepool | number | | The value must be between `min_nodes` and `max_nodes`|
| node_taints | Taints for the nodepool VMs | list of strings | | |
| node_labels | Labels to add to the nodepool VMs | map | | On nodes you wish to run SAS pods you will need to include this label: `"workload.sas.com/node"  = ""` |
| custom_data | Additional userdata that will be appended to the default userdata. | string | | The value must be an empty string "" or the path to a file containing a `bash` script snippet that will be executed on the node pool. |

## Storage

| <div style="width:50px">Name</div> | <div style="width:150px">Description</div> | <div style="width:50px">Type</div> | <div style="width:75px">Default</div> | <div style="width:150px">Notes</div> |
| :--- | :--- | :--- | :--- | :--- |
| storage_type | Type of Storage. Valid Values: "standard", "ha"  | string | "standard" | "standard" creates NFS server VM, "ha" creates an AWS EFS mountpoint |

### NFS Server

When `storage_type=standard`, a NFS Server VM is created and these variables are applicable.

<!--| Name | Description | Type | Default | Notes | -->
| <div style="width:50px">Name</div> | <div style="width:150px">Description</div> | <div style="width:50px">Type</div> | <div style="width:75px">Default</div> | <div style="width:150px">Notes</div> |
| :--- | :--- | :--- | :--- | :--- |
| create_nfs_public_ip | Add public ip to the NFS server VM | bool | false |  |
| nfs_vm_admin | OS Admin User for the NFS server VM | string | "nfsuser" | |
| nfs_raid_disk_size | Size in GiB for each EBS volume of the RAID0 cluster on the NFS server VM | number | 128 | |
| nfs_raid_disk_type | Disk type for the NFS server EBS volume | string | "gp2" | Valid values: "standard", "gp2", "io1", "io2", "sc1" or "st1" |
| nfs_raid_disk_iops | IOPS for the the NFS server EBS volumes | number | 0 | Only used when `nfs_raid_disk_type` is "io2" or "io2" |

### AWS Elastic File System (EFS)

When `storage_type=ha`, [AWS Elastic File System](https://aws.amazon.com/efs/) service is created and these variables are applicable.

<!--| Name | Description | Type | Default | Notes | -->
| <div style="width:50px">Name</div> | <div style="width:150px">Description</div> | <div style="width:50px">Type</div> | <div style="width:75px">Default</div> | <div style="width:150px">Notes</div> |
| :--- | :--- | :--- | :--- | :--- |
| efs_performance_mode | EFS performance mode | string | generalPurpose | Supported values are `generalPurpose` or `maxIO` |

## Postgres

<!--| Name | Description | Type | Default | Notes | -->
| <div style="width:50px">Name</div> | <div style="width:150px">Description</div> | <div style="width:50px">Type</div> | <div style="width:75px">Default</div> | <div style="width:150px">Notes</div> |
| :--- | :--- | :--- | :--- | :--- |
| create_postgres | Create an AWS Postgres DB (RDS) server instance | bool | false | |
| postgres_server_name | Name of PostgreSQL server | string | "" | Changing this value trigger resource recreation |
| postgres_server_version | The version of the PostgreSQL server | string | "11" | Changing this value trigger resource recreation |
| postgres_instance_type | The VM type for the PostgreSQL Server | string | "db.m5.xlarge" | |
| postgres_storage_size | Max storage allowed for the PostgreSQL server in MV | number | 50 |  |
| postgres_backup_retention_days | Backup retention days for the PostgreSQL server | number | 7 | Supported values are between 7 and 35 days. |
| postgres_storage_encrypted | Encrypt PostgrSQL data at rest | bool | false| |
| postgres_administrator_login | The Administrator Login for the PostgreSQL Server | string | "pgadmin" | Changing this forces a new resource to be created |
| postgres_administrator_password | The Password associated with the postgres_administrator_login for the PostgreSQL Server | string | | |
| postgres_db_name | Name of database to create | string | "SharedServices" | |
| postgres_multi_az | Specifies if PostgreSQL instance is multi-AZ | bool | false | |
| postgres_deletion_protection | Protect from accidental resource deletion | bool | false | |
| postgres_parameters | additional parameters for PostgreSQL server | list of maps | [] | |
| postgres_options | additional options for PostgreSQL server | list of maps | [] |   |
