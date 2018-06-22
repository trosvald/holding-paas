# holding-paas

## A. Prerequisites
#### 1. Master Hosts
In a highly available OpenShift Origin cluster with external etcd, a master host should have, in addition to the minimum requirements in the table above, 1 CPU core and 1.5 GB of memory for each 1000 pods. Therefore, the recommended size of a master host in an OpenShift Origin cluster of 2000 pods would be the minimum requirements of 2 CPU cores and 16 GB of RAM, plus 2 CPU cores and 3 GB of RAM, totaling 4 CPU cores and 19 GB of RAM.

>__When planning an environment with multiple masters, a minimum of three etcd hosts and a load-balancer between the master hosts are required.__
