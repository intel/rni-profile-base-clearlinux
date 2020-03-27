# Clear Linux OS Profile

<img align="right" src="https://avatars1.githubusercontent.com/u/12545216?s=200&v=4">

Intended to be used with Retail Node Installer and Clear Linux base profile, this Clear Linux OS profile contains a few files that ultimately will install Clear Linux OS with Retail Workload Orchestrator to disk.

## Software Stack in this profile

* Clear Linux
* Retail Workload Orchestrator

  
## Target Device Prerequisites

* x86 Bare Metal or x86 Virtual Machine
* At Least 5 GB of Disk Space
  * Supports the following drive types:
    * SDD
    * NVME
    * MMC
* 4 GB of RAM

## Getting Started

The necessary prerequisites to using this profile are as below
  * Retail Node Installer deployed. Please refer to Retail Node Installe project [documentation for installation](https://github.com/intel/retail-node-installer) in order to deploy it.
  * Build all [Retail Workload Orchestator service's](https://github.com/intel/RetailWorkloadOrchestrator/blob/master/build.sh) docker images mentioned in conf/files.sample.yml under private_docker_registry_images section. Push it to private docker registry and configure private docker registry details in files.yml to use it.

Out of the box, the Clear Linux profile should _just work_. Therefore, no specific steps are required in order to use this profile that have not already been described in the Retail Node Installer documentation. Simply boot a client device using legacy BIOS PXE boot and the Clear Linux profile should automatically launch after a brief waiting period.

If you do encounter issues PXE booting, please review the steps outlined in the Retail Node Installer documentation and ensure you've followed them correctly. See the [Known Issues](https://github.com/intel/retail-node-installer) section for possible solutions.

After installing Clear Linux, the default login username is `sys-admin` and the default password is `P@ssw0rd!`. This password is defined in the `bootstrap.sh` script and in the `conf/config.yml` as a kernel argument.

## Kernel Paramaters used at build time

The following kernel parameters can be added to `conf/config.yml`

* `bootstrap` - RESERVED, do not change
* `clearlinuxversion` - Use the release number from [here](https://cdn.download.clearlinux.org/releases/)  Defaults to latest release
* `debug` - [TRUE | FALSE] Enables a more verbose output
* `httppath` - RESERVED, do not change
* `kernparam` - Used to pass additional kernel parameters to the targeted system.  Example format: kernparam=splash:quiet#enable_gvt:1
* `parttype` - RESERVED, do not change
* `password` - Initial user password. Defaults to 'password'
* `proxy` - Add proxy settings if behind proxy during installation.  Example: <http://proxy-us.intel.com:912>
* `proxysocks` - Add socks proxy settings if behind proxy during installation.  Example: <http://proxy-us.intel.com:1080>
* `release` - [prod | dev] If set to prod the system will shutdown after it is provisioned.  Altnerativily it will reboot.
* `token` - GitHub token for private repositories, if this profile is in a private respository this token should have access to this repo
* `username` - Initial user name. Defaults to 'sys-admin'
* `rwo` - GitHub branch of Retail Workload Orchestrator repository. Defaults to 'master'

## Sample Profile Section

* To use base profile with custom profile, Please refer below sample profile section of config.yml for Retail Node Installer 

```yaml
# Please make sure to define ALL of the variables below, even if they
# are empty. Otherwise, this application will not be configured properly.
profiles:
  - git_remote_url: https://github.com/intel/rni-profile-base-clearinux.git
    profile_branch: rwo
    profile_base_branch: master
    git_username: ""
    git_token: ""
    name: clearlinux_profile
    custom_git_arguments: ""
```

## Known Limitations

* Currently does not support full disk encryption
* Currently does not install Secure Boot features

## Post Installation

* Please refer to Retail Workload Orchestrator project [documentation](https://github.com/intel/RetailWorkloadOrchestrator/blob/master/docs) in order to set up security and swarm management tool on node.

## Customization

If you want to customize your Retail Node Installer profile, follow these steps:

* Duplicate this repository locally and push it to a separate/new git repository
* Make changes after reading the information below
* Update your Retail Node Installer configuration to point to the git repository, base branch (such as master or base) and custom branch(such as rwo).

The flexibility of Retail Node Installer comes to fruition with the following profile-side file structures:

* `conf/config.yml` - This file contains the arguments that are passed to the Linux kernel upon PXE boot. Alter these arguments according to the needs of your scripts. The following kernel arguments are always prepended to the arguments specified in `conf/config.yml`:
  * `console=tty0`
  * `httpserver=@@HOST_IP@@`
  * `bootstrap=http://@@HOST_IP@@/profile/${profileName}/bootstrap.sh`
* `conf/files.yml` - This file contains a few definitions that tell Retail Node Installer to download specific files that you can customize. **Please check if there are any [Known Limitations](#Known-Limitations) before changing this file from the default.** User can specify an `initrd` and `vmlinuz`, as shown in the `conf/files.sample.yml` file. See `conf/files.sample.yml` for a full example.
* `bootstrap.sh` - A profile is required to have a `bootstrap.sh` as an entry point. This is an arbitrary script that you can control. Custom bootstrap.sh should always call pre.sh and post.sh from base branch inorder to install OS(Please refer *rwo* custom branch for reference). User can also write a seprate script(such as profile.sh) to perform specific task and call it from bootstrap.sh.

Currently the following variables are processed:
  * `@@DHCP_MIN@@`
  * `@@DHCP_MAX@@`
  * `@@NETWORK_BROADCAST_IP@@`
  * `@@NETWORK_GATEWAY_IP@@`
  * `@@HOST_IP@@`
  * `@@NETWORK_DNS_SECONDARY@@`
  * `@@PROFILE_NAME@@`

### Customization Requirements

A profile **must** have all of the following:

* a `bootstrap.sh` file at the root of the repository
