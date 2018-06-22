# holding-paas

## A. Prerequisites
#### 1. Master Hosts
In a highly available OpenShift Origin cluster with external etcd, a master host should have, in addition to the minimum requirements in the table above, 1 CPU core and 1.5 GB of memory for each 1000 pods. Therefore, the recommended size of a master host in an OpenShift Origin cluster of 2000 pods would be the minimum requirements of 2 CPU cores and 16 GB of RAM, plus 2 CPU cores and 3 GB of RAM, totaling 4 CPU cores and 19 GB of RAM.

>__When planning an environment with multiple masters, a minimum of three etcd hosts and a load-balancer between the master hosts are required.__

#### 2. Node Hosts
The size of a node host depends on the expected size of its workload. As an OpenShift Origin cluster administrator, you will need to calculate the expected workload, then add about 10 percent for overhead. For production environments, allocate enough resources so that a node host failure does not affect your maximum capacity.
## B. GlusterFS Hardware Requirements
Any nodes used in a Containerized GlusterFS or External GlusterFS cluster are considered storage nodes. Storage nodes can be grouped into distinct cluster groups, though a single node can not be in multiple groups. For each group of storage nodes:

- A minimum of three storage nodes per group is required.
- Each storage node must have a minimum of 8 GB of RAM. This is to allow running the GlusterFS pods, as well as other applications and the underlying operating system.
  - Each GlusterFS volume also consumes memory on every storage node in its storage cluster, which is about 30 MB. The total amount of RAM should be determined based on how many concurrent volumes are desired or anticipated.
- Each storage node must have at least one raw block device with no present data or metadata. These block devices will be used in their entirety for GlusterFS storage. Make sure the following are not present:
  - Partition tables (GPT or MSDOS)
  - Filesystems or residual filesystem signatures
  - LVM2 signatures of former Volume Groups and Logical Volumes
  - LVM2 metadata of LVM2 physical volumes
- If in doubt, ```wipefs -a <device>``` should clear any of the above.
>It is recommended to plan for two clusters: one dedicated to storage for infrastructure applications (such as an OpenShift Container Registry) and one dedicated to storage for general applications. This would require a total of six storage nodes. This recommendation is made to avoid potential impacts on performance in I/O and volume creation.

#### GlusterFS
To access GlusterFS volumes, the ```mount.glusterfs``` command must be available on all **__schedulable nodes__**. For RPM-based systems, the ```glusterfs-fuse``` package must be installed:
```
# yum install glusterfs-fuse
```
If ```glusterfs-fuse``` is already installed on the nodes, ensure that the latest version is installed:
```
# yum update glusterfs-fuse
```
## C. Host Preparation
#### 1. Installing Base Packages
- Install the following base packages:
```
# yum install -y wget git net-tools bind-utils yum-utils iptables-services bridge-utils bash-completion kexec-tools sos psacct
```
- Update the system to the latest packages:
```
# yum update
# systemctl reboot
```

#### 2. Installing Docker
>At this point, you should install Docker on all master and node hosts. This allows you to configure your **__Docker storage options__** before installing OpenShift Origin.
>
- For RHEL 7 systems, install Docker 1.13:
```
# yum install docker-1.13.1
```
- After the package installation is complete, verify that version 1.13 was installed:
```
# rpm -V docker-1.13.1
# docker version
```

#### 3. Configuring Thin Pool Storage for Docker
You can use the ```docker-storage-setup``` script included with Docker to create a thin pool device and configure Docker’s storage driver. This can be done after installing Docker and should be done before creating images or containers. The script reads configuration options from the ```/etc/sysconfig/docker-storage-setup``` file and supports three options for creating the logical volume:
- **Option A)** Use an additional block device.
- **Option B)** Use an existing, specified volume group.
- **Option C)** Use the remaining free space from the volume group where your root file system is located.

**Option A** is the most robust option, however it requires adding an additional block device to your host before configuring Docker storage. Options B and C both require leaving free space available when provisioning your host. Option C is known to cause issues with some applications, for example Red Hat Mobile Application Platform (RHMAP).

a. Create the docker-pool volume using one of the following three options:
  - **Option A)** Use an additional block device.

  In ```/etc/sysconfig/docker-storage-setup``` , set **DEVS** to the path of the block device you wish to use. Set **VG** to the volume group name you wish to create; ```docker-vg``` is a reasonable choice. For example:

  ```
  # cat <<EOF > /etc/sysconfig/docker-storage-setup
    DEVS=/dev/vdc
    VG=docker-vg
    EOF
  ```
  Then run ```docker-storage-setup``` and review the output to ensure the ``docker-pool`` volume was created.
