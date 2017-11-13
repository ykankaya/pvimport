# pvimport

## Version

Stable release: **v1.3.2**  
Testing release: **testing**

## Description

Allows you to automate all the work involved in importing virtual machine disks from the current hypervisor (**VMware ESXi/Xen**) to the **<u>Proxmox VE</u>** cluster.

Performs:

- execute a snapshot of the virtual machine and export it to an \*.ova file - **Xen**
- copying the virtual machine disks from the source (remote hypervisor) to the destination (local resource) - **Xen/VMware ESXi**
- extraction of disks with extension \*.ova (resulting directories: Ref:\*) - **Xen**
- convert all disks to the selected format (**raw**/**qcow2**) - **Xen/VMware ESXi**
- converted **raw**/**qcow2** files imports into place created when creating the virtual machine (**directory**/**lvm**) - also on the selected **Proxmox VE** node - **Xen/VMware ESXi**

## Parameters

Provides the following options:

``````
  Usage:
    pvimport <option|long-option>

  Examples:
    pvimport -c vmware.cfg -h pv01 -i gitlab_01 -p 300 -f raw --verbose
    pvimport -c xen.cfg -h 172.20.50.31 -i ac06d737 -p 200 -f qcow2 --pve-import local --pve-type dir

  Options:
        --help                      show this message
        --debug                     displays information on the screen (debug mode)
        --verbose                   displays 'info' messages on the screen (verbose mode)
        --time                      displays execution time, occurs only with --verbose
    -c, --config <file>             attach an external config file to the script
    -h, --host <host>               sets the ip address or hostname of the remote hypervisor
    -i, --id <vm_id|vm_name>        sets the remote vm id (Xen) or vm name (Xen/VMware ESXi)
    -p, --pve-id <num>              sets the vm id created in Proxmox VE
    -f, --pve-format <raw|qcow2>    sets the disk output format
        --pve-import <local|host>   import disks into any Proxmox VE node
        --pve-type <dir|lvm>        sets the target asset to which the disks will be imported
``````

If you want only converted files do not use the options below - they are optional:

- `--pve-import`
- `--pve-type`

## Configuration file

The configuration file (appended with the `-c|--config` parameter) has the following structure:

``````
# shellcheck shell=bash

# Specifies the type of hypervisor (VMware ESXi or Xen).
readonly hv_type="type"

# Specifies the port number through which the ssh connection
# to the remote server is established (ssh/scp). The ip address
# or hostname is determined by the parameter (-h|--host).
#   Example: port="22"
readonly port="22"

# Specifies the parameters for the ssh protocol. Before setting
# test whether the server accepts it.
#   Example: ssh_opt="-C -c arcfour -vv"
readonly ssh_opt=""

# Specifies the parameters for the dd command. Before setting
# test whether the server accepts it.
#   Example: dd_opt="bs=16M"
readonly dd_opt="bs=16M"

# Specifies the remote path on the remote machine where
# vm files will be placed. In this place will be created
# directory for Xen snapshots.
#   Example: hv_storage="/vmfs/volumes/datastore1"
readonly hv_storage="/path/to/remote/vm/dump"

# Specifies the local resource of virtual machines.
# Used only for memory specified as directory.
#   Example: pve_storage="/xfs900/images"
readonly pve_storage="/path/to/proxmox/images"

# Specifies the local resource of virtual machines.
# Used only for memory specified as LVM.
#   Example: pve_lvm="/dev/pve"
readonly pve_lvm="/path/to/lvm/vg"

# Specifies the local path (remember to create it) on the Proxmox VE machine
# where the virtual machine files from the remote host will be copied.
#   Example: local_storage="/xfs900/pvimport"
readonly local_storage="/path/to/local/vm/dump"

# Specifies whether to delete unneeded files/directories (only local).
#   Example: remove_unused="yes"
readonly remove_unused="no"
``````

## Before importing

- set the **key authorization** (**pvimport** uses ssh protocol for communication):
  - **Xen** (for root user): */root/.ssh/authorized_keys*
  - **VMware** (for root user): */etc/ssh/keys-root/authorized_keys*
- prepare the **correct configuration file** (*src/configs/template.cfg*)
- create **local/working directory** (details above)

## Requirements

**<u>Pvimport</u>** uses external utilities to be installed before running:

- [qemu-img](https://en.wikibooks.org/wiki/QEMU/Installing_QEMU)
- [xenmigrate](https://pve.proxmox.com/wiki/Xenmigrate) (only for **Xen**)

## Use example

> - before you start, create a virtual machine in the **Proxmox VE** web panel. The most important thing is to add the same number of disks of the same size as the current hypervisor
> - after creating a virtual machine from **Proxmox VE** web panel, the next step is to create a local working directory (in the configuration file: `local_storage`)

Then an example of starting the tool:

``````
pvimport -c src/configs/xen.cfg -h xen01 -i web01 -p 205 -f raw --pve-import local --pve-type dir --time --verbose
``````

In the first place we define the configuration (which should be prepared in advance):

- `-c src/configs/xen.cfg`

Specify hostname (in this example **Xen** hypervisor):

- `-h xen01`

Specify the registered virtual machine name - **uuid** parameter after issuing the `xe vm-list` command:

- `-i web01`

This parameter specifies the identifier under which the virtual machine will be visible from **Proxmox VE**:

- `-p 205`

Specifies the resulting format of the created files - available values are **raw** or **qcow2**:

- `-f raw`

Specifies the import target **Proxmox VE** node. It is a local node (from which we init **pvimport**) or remote node:

- `--pve-import local`

Specifies the import target resource. There are two types - **directory** or **lvm** (according to what disks we created for the virtual machine):

- `--pve-type dir`

Displays the execution time of the selected commands (only with `--verbose` mode):

- `--time`

Verbose mode - displays more detailed information on the screen:

- `--verbose`

## Important

- exporting a virtual machine running under **Xen** takes place by taking a **snapshot** which allows the virtual machine to run continuously (until the final import) - the disadvantage of this solution may be the current content of the disk
- **pvimport** can be run on **any Proxmox VE node**. Remember to **have enough space** for the output files in the **vmdk** and **raw/qcow2** format
- the `--pve-import <local|host>` parameter allows you to **import virtual machine files to any node** (not necessarily from which **pvimport** was started)

## Limitations

- does not create a virtual machine from **Proxmox VE** (cli/web) - you have to do it yourself
- requires a disk space of the same size as the imported virtual machine - to store all files (disks)
- hardware and network resources are the major constraints that affect the time of importing disks
- at the moment it does not support **vmdk** drives as target drives (only in **raw**/**qcow2** format)

## Project architecture

    |-- pvimport                # main script (init)
    |-- LICENSE.md              # GNU GENERAL PUBLIC LICENSE, Version 3, 29 June 2007
    |-- README.md               # this simple documentation
    |-- .gitignore              # ignore untracked files
    |-- .gitkeep                # track empty directory
    |-- src                     # includes external project files
        |-- _import_            # external variables and functions
        |-- vmware              # external configuration for VMware ESXI
        |-- xen                 # external configuration for Xen
        |-- configs             # directory with configurations
            |-- template.cfg    # template configuration
    |-- doc                     # includes documentation, images and manuals

## License

GPLv3 : <http://www.gnu.org/licenses/>

**Free software, Yeah!**
