# pure-csi

This helm chart installs the CSI plugin on a Kubernetes cluster.

## Platform and Software Dependencies
- #### Operating Systems Supported*:
  - CentOS 7
  - CoreOS (Ladybug 1298.6.0 and above)
  - RHEL 7
  - Ubuntu 16.04
  - Ubuntu 18.04
- #### Environments Supported*:
  - Kubernetes 1.13+
  - Minimum Helm version required is 2.9.1.
  - OpenShift 3.11
- #### Other software dependencies:
  - Latest linux multipath software package for your operating system (Required)
  - Latest Filesystem utilities/drivers (XFS by default, Required)
  - Latest iSCSI initiator software for your operating system (Optional, required for iSCSI connectivity)
  - Latest NFS software package for your operating system (Optional, required for NFS connectivity)
  - Latest FC initiator software for your operating system (Optional, required for FC connectivity)
- #### FlashArray and FlashBlade:
  - The FlashArray and/or FlashBlade should be connected to the compute nodes using [Pure's best practices](https://support.purestorage.com/Solutions/Linux/Reference/Linux_Recommended_Settings)

_* Please see release notes for details_

## Additional configuration for Kubernetes 1.13 Only
For details see the [CSI documentation](https://kubernetes-csi.github.io/docs/csi-driver-object.html). 
In Kubernetes 1.12 and 1.13 CSI was alpha and is disabled by default. To enable the use of a CSI driver on these versions, do the following:

1. Ensure the feature gate is enabled via the following Kubernetes feature flag: ```--feature-gates=CSIDriverRegistry=true```
2. Either ensure the CSIDriver CRD is installed cluster with the following command:
```
$> kubectl create -f https://raw.githubusercontent.com/kubernetes/csi-api/master/pkg/crd/manifests/csidriver.yaml
```
## CSI Snapshot and Clone features for Kubernetes
For details see the [CSI volume snapshot](https://kubernetes-csi.github.io/docs/snapshot-restore-feature.html) and [CSI volume clone](https://kubernetes-csi.github.io/docs/volume-cloning.html).
1. For snapshot feature, ensure you have Kubernetes 1.13+, the feature gate is enabled via the following Kubernetes feature flag: ```--feature-gates=VolumeSnapshotDataSource=true```
2. For clone feature, ensure you have Kubernetes 1.15+, Ensure the feature gate is enabled via the following Kubernetes feature flag: ```--feature-gates=VolumePVCDataSource=true```

## How to install

Add the Pure Storage helm repo
```
helm repo add pure https://purestorage.github.io/helm-charts
helm repo update
helm search pure-csi
```

Optional (offline installation): Download the helm chart
```
git clone https://github.com/purestorage/helm-charts.git
```

Create your own values.yaml and install the helm chart with it, and keep it. Easiest way is to copy
the default [./values.yaml](./values.yaml).

### Configuration
The following table lists the configurable parameters and their default values.

|             Parameter       |            Description             |                    Default                |
|-----------------------------|------------------------------------|-------------------------------------------|
| `image.name`                | The image name       to pull from  | `purestorage/k8s`                         |
| `image.tag`                 | The image tag to pull              | `5.0.2`                                   |
| `image.pullPolicy`          | Image pull policy                  | `IfNotPresent`                            |
| `app.debug`                 | Enable/disable debug mode for app  | `false`                                   |
| `storageclass.isPureDefault`| Set `pure` storageclass to the default | `false`                               |
| `storageclass.pureBackend`  | Set `pure` storageclass' default backend type | `block`                               |
| `clusterrolebinding.serviceAccount.name`| Name of K8s/openshift service account for installing the plugin | `pure`                    |
| `flasharray.defaultFSType`  | Block volume default filesystem type. *Not recommended to change!* | `xfs`     |
| `flasharray.defaultFSOpt`  | Block volume default mkfs options. *Not recommended to change!* | `-q`          |
| `flasharray.defaultMountOpt`  | Block volume default filesystem mount options. *Not recommended to change!* |     ""    |
| `flasharray.iSCSILoginTimeout`  | iSCSI login timeout in seconds. *Not recommended to change!* |     `20sec`    |
| `flasharray.iSCSIAllowedCIDR`  | List of CIDR blocks allowed as iSCSI targets, e.g. 10.0.0.0/24,10.1.0.0/16. Use comma (,) as the separator, and empty string means allowing all addresses. |     ""    |
| `flasharray.preemptAttachments`  | Enable/Disable attachment preemption! |     `true`    |
| `flasharray.sanType`        | Block volume access protocol, either ISCSI or FC | `ISCSI`                     |
| `flashblade.snapshotDirectoryEnabled`  | Enable/Disable FlashBlade snapshots |     `false`    |
| `namespace.pure`            | Namespace for the backend storage  | `k8s`                                     |
| `orchestrator.name`         | Orchestrator type, such as openshift, k8s | `k8s`                              |
| *`arrays`                    | Array list of all the backend FlashArrays and FlashBlades | must be set by user, see an example below                |
| `mounter.nodeSelector`              | [NodeSelectors](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector) Select node-labels to schedule CSI node plugin. | `{}` |
| `mounter.tolerations`               | [Tolerations](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/#concepts)  | `[]` |
| `mounter.affinity`                  | [Affinity](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity) | `{}` |
| `provisioner.nodeSelector`              | [NodeSelectors](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector) Select node-labels to schedule provisioner. | `{}` |
| `provisioner.tolerations`               | [Tolerations](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/#concepts)  | `[]` |
| `provisioner.affinity`                  | [Affinity](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity) | `{}` |
| `csi.provisioner.image.name` | The image name of the csi-provisioner | `quay.io/k8scsi/csi-provisioner` |
| `csi.provisioner.image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `csi.clusterDriverRegistrar.image.name` | The image name of the csi-cluster-driver-registrar | `quay.io/k8scsi/csi-cluster-driver-registrar` |
| `csi.clusterDriverRegistrar.image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `csi.nodeDriverRegistrar.image.name` | The image name of the csi-node-driver-registrar | `quay.io/k8scsi/csi-node-driver-registrar` |
| `csi.nodeDriverRegistrar.image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `csi.livenessProbe.image.name` | The image name of the csi livenessprobe | `quay.io/k8scsi/livenessprobe` |
| `csi.livenessProbe.image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `csi.snapshotter.image.name` | The image name of the csi snapshotter | `quay.io/k8scsi/csi-snapshotter` |
| `csi.snapshotter.image.pullPolicy` | Image pull policy | `IfNotPresent` |

*Examples:
```yaml
arrays:
  FlashArrays:
    - MgmtEndPoint: "1.2.3.4"
      APIToken: "a526a4c6-18b0-a8c9-1afa-3499293574bb"
      Labels:
        rack: "22"
        env: "prod"
    - MgmtEndPoint: "1.2.3.5"
      APIToken: "b526a4c6-18b0-a8c9-1afa-3499293574bb"
  FlashBlades:
    - MgmtEndPoint: "1.2.3.6"
      APIToken: "T-c4925090-c9bf-4033-8537-d24ee5669135"
      NfsEndPoint: "1.2.3.7"
      Labels:
        rack: "7b"
        env: "dev"
    - MgmtEndPoint: "1.2.3.8"
      APIToken: "T-d4925090-c9bf-4033-8537-d24ee5669135"
      NfsEndPoint: "1.2.3.9"
      Labels:
        rack: "6a"
```

## Assigning Pods to Nodes

It is possible to make CSI Node Plugin and CSI Controller Plugin to run on specific nodes
using `nodeSelector`, `toleration`, and `affinity`. You can set these config
separately for Node Plugin and Controller Plugin using `mounter.nodeSelector`, and `provisioner.nodeSelector` respectively.

## Install the plugin in a separate namespace(i.e. project)
For security reason, it's strongly recommended to install the plugin in a separate namespace/project. Make sure the namespace is existing, otherwise create it before installing the plugin.

Customize your values.yaml including arrays info (replacement for pure.json), and then install with your values.yaml.

Dry run the installation, and make sure your values.yaml is working correctly.
```
helm install --name pure-storage-driver pure/pure-csi --namespace <namespace> -f <your_own_dir>/yourvalues.yaml --dry-run --debug
```

Run the Install
```
# Install the plugin 
helm install --name pure-storage-driver pure/pure-csi --namespace <namespace> -f <your_own_dir>/yourvalues.yaml
```

The values in your values.yaml overwrite the ones in pure-csi/values.yaml, but any specified with the `--set`
option will take precedence.
```
helm install --name pure-storage-driver pure/pure-csi --namespace <namespace> -f <your_own_dir>/yourvalues.yaml \
            --set flasharray.sanType=fc \
            --set namespace.pure=k8s_xxx \
```

## How to update `arrays` info

Update your values.yaml with the correct arrays info, and then upgrade the helm as below.

**Note**: Ensure that the values for `--set` options match when run with the original install step. It is highly recommended
to use the values.yaml and not specify options with `--set` to make this easier.
```
helm upgrade pure-storage-driver pure/pure-csi --namespace <namespace> -f <your_own_dir>/yourvalues.yaml --set ...
```

# Upgrading
## How to upgrade the driver version

It's not recommended to upgrade by setting the `image.tag` in the image section of values.yaml. Use the version of
the helm repository with the tag version required. This ensures the supporting changes are present in the templates.
```
# list the avaiable version of the plugin
helm repo update
helm search pure-csi -l

# select a target chart version to upgrade as
helm upgrade pure-storage-driver pure/pure-csi --namespace <namespace> -f <your_own_dir>/yourvalues.yaml --version <target chart version>
```

## How to upgrade from the flexvolume to CSI

Upgrade from flexvolume to CSI is not currently supported and is being considered for an upcoming release.

# Release Notes
Release notes can be found [here](https://github.com/purestorage/helm-charts/releases)

### Known Vulnerabilities 
None


# License
https://www.purestorage.com/content/dam/pdf/en/legal/pure-storage-plugin-end-user-license-agreement.pdf
