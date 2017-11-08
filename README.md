# pvimport

## Version

Stable release: **v1.3.1**  
Testing release: **testing**

## Description

Allows you to automate all the work involved in importing virtual machine disks from the current hypervisor (**VMware ESXi/Xen**) to the **<u>Proxmox VE</u>** cluster.

Performs:

- execute a snapshot of the virtual machine and export it to an \*.ova file - **Xen**
- extraction of disks with extension \*.ova (resulting directories: Ref:\*) - **Xen**
- copying the virtual machine disks from the source (remote hypervisor) to the destination (local resource, working directory) - **Xen/VMware**
- convert to the selected format (img/qcow2) - **Xen/VMware**
- converted img/qcow2 files imports into place created when creating the virtual machine (directory/lvm) also on the selected proxmox node - **Xen/VMware**

## Parameters

The tool provides the following options:

``````
  Usage:
    pvimport <option|long-option>

  Examples:
    pvimport -c vmware.cfg -h pv01 -i gitlab_01 -p 300 -f img --verbose
    pvimport -c xen.cfg -h 172.20.50.31 -i ac06d737 -p 200 -f qcow2 --pve-import local

  Options:
        --help                      show this message
        --debug                     display information on the screen (debug mode)
        --verbose                   display 'info' messages on the screen (verbose mode)
        --time                      displays the total execution time of the script
    -c, --config <file>             attach an external config file to the script
    -h, --host <host>               sets the ip address or hostname of the remote hypervisor
    -i, --id <vm_id|vm_name>        sets the remote id (Xen) or name (Xen/VMware ESXi) of the imported vm
    -p, --pve-id <num>              sets the vm id created in proxmox
    -f, --pve-format <img|qcow2>    sets the disk output format
        --pve-import <local|host>   import disks into any proxmox node (optional)
``````

## Configuration file

The configuration file (appended with the `-c|--config` parameter) has the following structure:

``````
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

# Specifies the remote path (remember to create it) on the remote machine
# where files (such as snapshots) will be placed (only for Xen).
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

# Specifies the local path (remember to create it) on the proxmox machine
# where the virtual machine files from the remote host will be copied.
#   Example: local_storage="/xfs900/vmware"
readonly local_storage="/path/to/local/vm/dump"

# Specifies whether to delete unneeded files/directories (only local).
#   Example: remove_unused="yes"
readonly remove_unused="no"

``````

## Before importing

- set the **key authorization** (**<u>pvimport</u>** uses ssh protocol for communication):
  - **Xen** (for root user): */root/.ssh/authorized_keys*
  - **VMware** (for root user): */etc/ssh/keys-root/authorized_keys*
- prepare the **correct configuration file** (*src/configs/template.cfg*)
- create **remote and local directory** (details above)

## Requirements

**<u>Pvimport</u>** uses external utilities to be installed before running:

- [qemu-img](https://en.wikibooks.org/wiki/QEMU/Installing_QEMU)
- [xenmigrate](https://pve.proxmox.com/wiki/Xenmigrate) (only for **Xen**)

## Use example

> Before you start, create a virtual machine in the proxmox web panel. The most important thing is to add the same number of disks of the same size as the current hypervisor.

``````
pvimport -c src/configs/xen.cfg -h xen01.domain.com -i web01 -p 205 -f img --verbose
``````

In the first place we define the configuration (which should be prepared in advance):

- `-c src/configs/xen.cfg`

Specify hostname (in this example xen hypervisor):

- `-h xen01.domain.com`

Specify the registered virtual machine name - **uuid** parameter after issuing the `xe vm-list` command:

- `-i web01`

This parameter specifies the identifier under which the virtual machine will be visible from proxmox (it is recommended to create a virtual machine first). This parameter is also very important from the standpoint of the `--sync` option, which syncs prepared disks with existing ones (after creating vm from proxmox):

- `-p 205`

Specifies the resulting format of the created files, available values are **img** (raw) or **qcow2**:

- `-f img`

Verbose mode - displays more detailed information on the screen:

- `--verbose`

## Important

- exporting a virtual machine running under **Xen** takes place by taking a **snapshot**, which allows the virtual machine to run continuously (until the final import) - the disadvantage of this solution may be the current content of the disk
- before exporting the virtual machine running under **VMware**, you must **remove all snapshots** - **<u>pvimport</u>** recognizes only the appropriate virtual machine (including flat) disks, further shortening the migration time
- **<u>pvimport</u>** can be run on **any proxmox node**. Remember **to have enough space** for the output files in the **img/qcow2** format (which when selected `--import <local|host>` will be deleted)
- the `--import <local|host>` parameter allows you to **import virtual machine files to any node** (not necessarily from which **<u>pvimport</u>** was started).

## Limitations

- does not create a virtual machine from proxmox (cli/web) - you have to do it yourself
- requires a disk space of the same size as the imported virtual machine - to store all files (disks)
- hardware and network resources are the major constraints that affect the time of importing disks

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
