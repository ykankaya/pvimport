# pvimport

## Description

Allows you to import virtual machines (disks) running under Vmware or Xen to Proxmox.

Performs:

- execute a snapshot of the virtual machine and export it to an ova file - **only for xen**

- copying the virtual machine from the source (remote hypervisor) to the destination (local resource) - **xen/vmware**

- extraction of disks with extension ova (resulting directories: Ref:\* -, **only for xen**

- convert to the selected format (img/qcow2) - **xen/vmware**

- converted img/qcow2 files imports into place created when creating the virtual machine (directory/lvm) - **xen/vmware**

Important:

> In this version, pvimport leaves all files/directories (ova file, Ref: directories).

## Script parameters

The tool provides the following options:

``````
  Usage:
    pvimport <option|long-option>

  Examples:
    pvimport -c xen.cfg -h 172.20.50.31 -i ac06d737 -n VM_PROD -p 200 -f qcow2 -s --verbose
    pvimport -c vmware.cfg -h 172.20.50.32 -i gitlab_01 -n gitlab_01 -p 300 -f img

  Options:
        --help                  show this message
        --version               show script version
        --debug                 display information on the screen (debug mode)
        --verbose               display 'info' messages on the screen
    -c, --config <file>         attach an external config file to the script
    -h, --host <ip|hostname>    sets the ip address or hostname of the remote hypervisor
    -i, --id <vm_id|vm_name>    sets the id (xen) or name (vmware) of the imported virtual machine
    -n, --name <new_vm_name>    sets a new name (xen/vmware) or directory name on the remote disk (vmware)
    -p, --pvid <num>            sets the virtual machine id created in proxmox
    -f, --format <img|qcow2>    sets the disk format (img or qcow2)
    -s, --sync                  synchronizes the created disks with the current ones (only for qcow2 format)
``````

## Config file

The configuration file (appended with the `-c|--config` parameter) has the following structure:

``````
# Specifies the type of hypervisor (xen or vmware).
readonly r_hypervisor_type="type"

# Specifies the user and the port number through which the connection
# to the remote server is established.
# The ip address or hostname is determined by the parameter (-h|--host).
readonly r_user="user"
readonly r_port="port"

# Specifies the remote path (remember to create it) on the remote machine
# where files (such as snapshots) will be placed.
readonly r_storage="/path/to/remote/vm/dump"

# Specifies the local path (remember to create it) on the proxmox machine
# where the virtual machine files from the remote host will be copied.
readonly l_storage="/xfs900/path/to/local/vm/dump"

# Specifies the local resource of virtual machines.
# Used only for memory specified as directory.
readonly l_pv_storage="/xfs900/images"

# Specifies the local resource of virtual machines.
# Used only for memory specified as LVM.
readonly l_pv_lvm="/dev/pve"
``````

## Example

``````
pvimport -c src/configs/xen.cfg -h xen01.domain.com -i web01 -n web01 -p 205 -f img --verbose
``````

In the first place we define the configuration (which should be prepared in advance):

- `-c src/configs/xen.cfg`

Specify host name (xen hypervisor):

- `-h xen01.domain.com`

Specify the registered virtual machine name - **uuid** parameter after issuing the `xe vm-list` command:

- `-i web01`

In the next step, we specify the name of the directory created in the local resource where the virtual machine will be visible (for xen, the value of `-i|--id` is the most important). For consistency, the value of this parameter may be the same as above:

- `-n web01`

This parameter specifies the identifier under which the virtual machine will be visible from proxmox (it is recommended to create a virtual machine first). This parameter is also very important from the standpoint of the `--sync` option, which syncs prepared disks with existing ones (after creating vm from proxmox):

- `-p 205`

Specifies the resulting format of the created files, available values are **qcow2** or **img** (raw):

- `-f img`

Verbose mode - displays more detailed information on the screen:

- `--verbose`

## Important

- exporting a virtual machine running under **xen** takes place by taking a **snapshot**, which allows the virtual machine to run continuously (until the final import) - the disadvantage of this solution may be the current content of the disk
- before exporting the virtual machine running under **vmware**, you must remove all snapshots - pvimport recognizes only the appropriate virtual machine (including flat) disks, further shortening the migration time

## Version

Latest stable: **v1.1.48**  
Devel branch: **next-release**

## Project architecture

    |-- LICENSE.md      # GNU GENERAL PUBLIC LICENSE, Version 3, 29 June 2007
    |-- README.md       # this simple documentation
    |-- .gitignore      # ignore untracked files
    |-- .gitkeep        # track empty directory
    |-- src             # includes external project files
    |-- doc             # includes documentation, images and manuals

## License

GPLv3 : <http://www.gnu.org/licenses/>

**Free software, Yeah!**
