# whereabouts Package

This package provides the ability to assign IP addresses dynamically across your Kubernetes cluster using a CNI IPAM plugin named [whereabouts](https://github.com/k8snetworkplumbingwg/whereabouts).

## Components

* Whereabouts Custom Resources
* Whereabouts DaemonSet
* Whereabouts ServiceAccount
* Whereabouts ClusterRoleBinding
* Whereabouts ip-reconciler Cronjob

## Configuration

The following configuration values can be set to customize the whereabouts installation.

### Global

| Value | Required/Optional | Description |
|-------|-------------------|-------------|
| `namespace` | Optional | The namespace in which to deploy whereabouts components. Default: kube-system |

### whereabouts Configuration

| Value | Required/Optional | Description |
|-------|-------------------|-------------|
| `whereabouts.config.resources.limits.cpu` | Optional | The limits for CPU resources of whereabouts DeamonSet  |
| `whereabouts.config.resources.limits.memory` | Optional | The limits for memory resources of whereabouts DeamonSet  |
| `whereabouts.config.resources.requests.cpu` | Optional | The requests for CPU resources of whereabouts DeamonSet  |
| `whereabouts.config.resources.requests.memory` | Optional | The requests for memory resources of whereabouts DeamonSet  |
| `ip_reconciler.config.schedule` | Optional | The schedule of ip-reconciler CronJob. Default: \*/5 \* \* \* \*  |
| `ip_reconciler.config.resources.requests.cpu` | Optional | The requests for memory resources of whereabouts DeamonSet  |
| `ip_reconciler.config.resources.requests.memory` | Optional | The requests for memory resources of whereabouts DeamonSet  |

## Usage Example

The follow is a basic guide for getting started with whereabouts.

This example guides you about attaching the second network interface to a pod with IP address assigned in the range you specified using whereabouts.

1. Install TCE Multus CNI package to support multiple network by following
   [doc](https://github.com/vmware-tanzu/community-edition/blob/main/addons/packages/multus-cni/3.7.1/README.md#usage-example):

1. Install TCE whereabouts package through tanzu CLI

    ```bash
    tanzu package install whereabouts --package-name whereabouts.community.tanzu.vmware.com --version ${MULTUS_PACKAGE_VERSION}
    ```

    > You can get the `${MULTUS_PACKAGE_VERSION}` from running `tanzu package
    > available list whereabouts.community.tanzu.vmware.com`. Specifying a
    > namespace may be required depending on where your package repository was
    > installed.

1. After the Multus CNI DaemonSet is running, you can define your network-attachment-defs to tell Multus CNI which CNI will be used for other network interfaces:

   ```bash
   cat <<EOF | kubectl create -f -
    apiVersion: "k8s.cni.cncf.io/v1"
    kind: NetworkAttachmentDefinition
    metadata:
    name: macvlan-conf-1
    spec:
    config: '{
        "cniVersion": "0.3.0",
        "name": "macvlan-conf-1",
        "type": "macvlan",
        "master": "eth0",
        "mode": "bridge",
        "ipam": {
            "type": "whereabouts",
            "range": "192.168.20.0/24",
            "gateway": "192.168.20.1",
            "range_start": "192.168.20.2",
            "range_end": "192.168.20.100"
        }
        }'
    EOF
    ```
