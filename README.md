k8s-deployment
==============

This repository contains a canonical Kubernetes cluster, as
deployed by the [k8s-boshrelease][1].  It provides a good starter
deployment, as well as operations files for adding new features,
scaling / renaming / resizing the cluster, and more.

To use this deployment, you will need a BOSH director, the
deployment of which is beyond the scope of this README and this
project.

Ops Files
=========

This repository contains a set of _ops files_ that adjust the base
manifest, allowing you to tweak, reconfigure, resize, and scale
your cluster deployment with ease.


### Renaming the Cluster / Deployment

```
bosh deploy k8s.yml \
  -o ops/rename.yml \
  -v name=shiny-new-cluster
```

By default, this deployment deploys to BOSH as the "k8s" deployment.
To change that, include this ops file and set the "name" variable to
whatever you would like to name the deployment.  Don't forget to also
change your `-d` flag to `bosh` CLI!

**Prerequisites**: None

**Variables**:

| Name   | Type   | Purpose |
| ------ | ------ | ------- |
| `name` | string | The new name for your cluster and BOSH deployment. |


### Scaling Control Plane and Worker Nodes

```
bosh deploy k8s.yml \
  -o ops/scale.yml \
  -v control_plane_instances=3 \
  -v worker_instances=8
```

Out of the box, this deployment gives you a single control
plane node (etcd + apiserver) and three worker nodes.

This ops file lets you scale those two instance groups
independently.  Best practices dictate that control nodes
always be deployed in odd numbers (usually 1, 3, or 5).
Worker nodes should be scaled to meet your workload demands.

**Prerequisites**: None

**Variables**:

| Name                     | Type   | Purpose |
| ------------------------ | ------ | ------- |
| `control_plan_instances` | number | How many control plane nodes to spin.  This should be an odd number, probably 1, 3 or 5.  Default is 1. |
| `worker_instances`       | number | How many worker nodes to spin.  Defaults to 3. |


### Resizing VMs (CPU / RAM / ephemeral storage)

```
bosh deploy k8s.yml \
  -o ops/resize_vms.yml \
  -v control_plane_vm_type=default \
  -v worker_vm_type=huge
```

Out of the box, both the "control" and "worker" instance_groups
will use the "default" VM type in your director's cloud config.

This ops file lets you override those, independently if you wish,
to use new VM types.

**Prerequisites**: None

**Variables**:

| Name                    | Type   | Purpose |
| ----------------------- | ------ | ------- |
| `control_plane_vm_type` | string | VM type (from cloud config) for the control plane nodes. |
| `worker_vm_type`        | string | VM type (from cloud config) for the worker nodes. |


### Resizing Persistent VM Disks

```
bosh deploy k8s.yml \
  -o ops/resize_disks.yml \
  -v control_plane_disk_size=40_000 \
  -v worker_disk_size=80_000
```

By default, control and worker nodes will be deployed
with 20Gi persistent disks, for storing generated state
data (like certificates we'd rather not re-generate).

This ops file lets you resize those persistent disks
independently.

**Prerequisites**: None

**Variables**:

| Name                      | Type | Purpose |
| ------------------------- | ---- | ------- |
| `control_plane_disk_size` | size | Size of the control plane node persistent volumes, in megabytes.  For the default, manually specify `20_000`. |
| `worker_disk_size`        | size | Size of the worker node persistent volumes, in megabytes.  For the default, manually specify `20_000`. |


### Using Latest (Uploaded) Release

```
bosh deploy k8s.yml \
  -o ops/latest-release.yml
```

This deployment recipe is pinned to a specific version of the
k8s BOSH release.  If, for whatever reason, you want or need to
use a newer version of the release, this ops file lets you pin
to the "latest" available on the BOSH director you are deploying
this cluster to.

Note that other issues with the manifest may crop up if your
targeted release version is too far in the past or the future.

**Prerequisites**: None

**Variables**: None


### Using Latest (Uploaded) Stemcell

```
bosh deploy k8s.yml \
  -o ops/latest-stemcell.yml
```

This deployment recipe is pinned to a specific version of Xenial
Ubuntu stemcell.  If, for whatever reason, you want or need to
use a newer version of the stemell, this ops file lets you pin
to the "latest" available on the BOSH director you are deploying
this cluster to.

**Prerequisites**: None

**Variables**: None


### Load Balancing

```
bosh deploy k8s.yml \
  -o ops/vip.yml \
  -v vrrp_id=128 \
  -v vip=10.0.254.7
```

This ops file co-locates a new load balancer job ("lb") on the control
node, and activates the keepalived VRRP feature to enable a single
entrypoint to a cluster of API server nodes.  For this to work, you
must choose a virtual router ID that is not in use on the local LAN,
as well as a virtual IP (the "VIP") to be the singular entrypoint.

**Prerequisites**: None

**Variables**:

| Name      | Type       | Purpose |
| --------- | ---------- | ------- |
| `vrrp_id` | number     | The Virtual Router ID, for the VRRP protocol.   This mus be unique within your network segment.  See [RFC 3768][rfc3768] for details. |
| `vip`     | IP address | The virtual IP to load balance via `keepalived`. |


### Alternate BOSH Network

```
bosh deploy k8s.yml \
  -o ops/network.yml \
  -v network_name=other_net
```

Out of the box, your Kubernetes cluster will be deployed
to the "default" network.  If you want to move it elsewhere,
this ops file patches up the network definitions, as well as
other references (namely the kubelet "instance-groups"
property) to match.

**Prerequisites**: None

**Variables**:

| Name           | Type   | Purpose |
| -------------- | ------ | ------- |
| `network_name` | string | The name of the BOSH network to use. |


### Bootstrapping Your Cluster With Additional Kubernetes Resources

```
bosh deploy k8s.yml \
  -o ops/bootstrap.yml \
  -v bootstrap_resources_yaml="... yaml ..."
```

This ops file lets you splice in your own (stringified) YAML
bootstrap Kubernetes resources, to be applied to your cluster
during post-deploy.

Note that if you introduce a syntax error in the YAML, the
deployment will fail.

**Prerequisites**: None

**Variables**:

| Name                       | Type   | Purpose |
| -------------------------- | ------ | ------- |
| `bootstrap_resources_yaml` | string | A YAML stream (stringified) to `kubectl apply` after the cluster has finished deploying. |


### NFS Capabilities

```
bosh deploy k8s.yml \
  -o ops/nfs.yml
```

This ops file enables the NFS kernel packages necessary
to allow Pods to mount remote Network File System mountpoints.

**Prerequisites**: None

**Variables**: None


### vSphere Cloud Provider

```
bosh deploy k8s.yml \
  -o ops/cloud/vsphere.yml \
  -v vsphere_ip=10.128.7.9 \
  -v vsphere_username=admin \
  -v vsphere_password=sekrit \
  -v vsphere_datacenter=dc1 \
  -v vsphere_datastore=fast-ssd \
  -v vsphere_network="VM Network" \
  -v vsphere_folder=/k8s/volumes
```

This ops file enables the vSphere Cloud Provider, so that your
Kubernetes cluster can provision VMDK assets in vCenter in
response to PersistentVolumeClaim requests for volumes to attach
to pods.

**Prerequisites**: None

**Variables**:

| Name                 | Type       | Purpose |
| -------------------- | ---------- | ------- |
| `vsphere_ip`         | IP address | IP address or hostname of your vSphere vCenter endpoint. |
| `vsphere_username`   | string     | Username for accessing vCenter's API. |
| `vsphere_password`   | string     | Password for authenticating to vCenter. |
| `vsphere_datacenter` | string     | Name of the vSphere data center you deployed the cluster to. |
| `vsphere_datastore`  | string     | Name of the datastore to provision volumes in. |
| `vsphere_network`    | string     | Name of a vSphere network for creating temporary VMs. |
| `vsphere_folder`     | string     | Folder (under the chosen datastore) where volumes should be provisioned. |


### vSphere Cloud Provider - Insecure / Skip TLS Verification

```
bosh deploy k8s.yml \
  -o ops/cloud/vsphere.yml \
  -o ops/cloud/vsphere-insecure.yml \
  -v vsphere_ip=10.128.7.9 \
  etc.
```

This ops file turns off vCenter TLS X.509 Certificate validation,
which may be required if your vCenter is using a self-signed
certificate, or a certificate from an unknown and/or untrusted
certificate authority.

**USE THIS OPS FILE WITH CAUTION.**

It has the potential to introduce man-in-the-middle attacks at
a *very* low level, with drastic repercussions.

**Prerequisites**: You must activate the `ops/cloud/vsphere.yml` ops file
first, to configure the rest of the vSphere Cloud Provider.

**Variables**: None (except those required by prerequisites).


### vSphere Cloud Provider - Custom Controller Type

```
bosh deploy k8s.yml \
  -o ops/cloud/vsphere.yml \
  -o ops/cloud/vsphere-custom-controller-type.yml \
  -v vsphere_ip=10.128.7.9 \
  etc. \
  -v vsphere_controller_type=whatever
```

This ops files lets you override the default volume controller
type with a custom value.

**Prerequisites**: You must activate the `ops/cloud/vsphere.yml` ops file
first, to configure the rest of the vSphere Cloud Provider.

**Variables** (in addition to those required by prerequisites):

| Name                      | Type   | Purpose |
| ------------------------- | ------ | ------- |
| `vsphere_controller_type` | string | The controller type to use instead of the default of "pvscsi" |


Contributing
============

See the [Code of Conduct][2] and [Contributing Guidelines][3].

[1]: https://github.com/jhunt/k8s-boshrelease
[2]: CONDUCT.md
[3]: CONTRIBUTING.md
[rfc3768]: https://tools.ietf.org/html/rfc3768
