# pvimport

## Description

Allows you to import virtual machines (disks) running under Vmware or Xen.

Performs:

- execute a snapshot of the virtual machine and export it to an ova file, **only for xen**

- copying the virtual machine from the source (remote hypervisor) to the destination (local resource), **xen/vmware**

- extraction of disks with extension ova (resulting directories: Ref:\*), **only for xen**

- convert to the selected format (img/qcow2), **xen/vmware**

- converted img/qcow2 files imports into place created when creating the virtual machine, **xen/vmware**

  > At this moment, only the disk resource mounted as a directory is supported. LVM is not support yet.

## Version

Latest stable: **v1.1**  
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
