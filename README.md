# profile_slurm

![pdk-validate](https://github.com/ncsa/puppet-profile_slurm/workflows/pdk-validate/badge.svg)
![yamllint](https://github.com/ncsa/puppet-profile_slurm/workflows/yamllint/badge.svg)

NCSA Customizations for Slurm

## Table of Contents

1. [Description](#description)
1. [Setup](#setup)
1. [Usage](#usage)
1. [Dependencies](#dependencies)
1. [Reference](#reference)


## Description

This is a module to add NCSA customizations to a Slurm client/scheduler/monitor node. It does not do the initial install/config of Slurm, that is handled by other modules (see [treydock/slurm](https://forge.puppet.com/modules/treydock/slurm) and [treydock/slurm_providers](https://forge.puppet.com/modules/treydock/slurm_providers) )

Currently this module:
- Adds more fine grained controls for firewalls, so will want to set `slurm::manage_firewall: false` from treydock/slurm to `false`
- Allows telegraf scripts to be deployed which collect Slurm metrics.


## Setup

For a Slurm submit node (generally cluster login nodes):
```
include ::profile_slurm::client
```

For a Slurm compute node:
```
include ::profile_slurm::client
include ::profile_slurm::compute
```

For a Slurm scheduler:
```
include ::profile_slurm::scheduler
```

For a Slurm monitor node (Include this on the node that hosts the `slurmdb`):
```
include ::profile_slurm::monitor
```


## Usage

You will want to set these hiera variables for Slurm submit nodes (generally cluster login nodes):
```
profile_slurm::client::firewall::sources:
  - "192.168.0.0/24"  # Allow access to slurmd from 192.168.0.0/24 net
```

You will generally want to manage the firewall as well as dependencies on local storage for Slurm compute nodes:
```
profile_slurm::client::firewall::sources:
  - "192.168.0.0/24"  # Allow access to slurmd from 192.168.0.0/24 net
profile_slurm::compute::storage::storage_dependencies:
  - "Mount['/local']"
profile_slurm::compute::storage::tmpfs_dir: "/local/slurmjobs"
profile_slurm::compute::storage::tmpfs_dir_refreshed_by: "Mount[/local]"
```

You will want to set these hiera variables for scheduler nodes:
```yaml
profile_slurm::scheduler::firewall::sources:
  - "192.168.0.0/24"  # Allow access to slurmdbd and slurmctld from 192.168.0.0/24 net
# and generally specify dependencies on local storage:
profile_slurm::scheduler::storage::storage_dependencies:
  - "Mount['/var/lib/mysql']"
  - "Mount['/var/log/slurm']"
  - "Mount['/var/spool/slurmctld.state']"
```

### Telegraf Monitoring

You will want to set these hiera variables for a node running the telegraf monitoring:
```yaml
profile_slurm::telegraf::telegraf::slurm_job_table: ""
```

If using MySQL ***with*** socket authentication, you'll also need to set the following because the `telegraf` user cannot use socket auth to connect as another user:
```yaml
profile_slurm::telegraf::slurm_username: "telegraf"
```

If using MySQL ***without*** socket authentication, you'll also need to set the following to lookup the password:
```yaml
profile_slurm::telegraf::telegraf::slurm_password: "%{lookup('slurm::slurmdbd_storage_pass')}"  # This is a VAULT lookup, use the keyname you have chosen for storing the slurmdb user account password
```


## Dependencies

n/a

## Reference

See: [REFERENCE.md](REFERENCE.md)

