topolvm-node
============

`topolvm-node` provides a CSI node service.  It also works as a custom
Kubernetes controller to implement dynamic volume provisioning.

CSI node features
-----------------

`topolvm-node` implements following optional features:

- [`GET_VOLUME_STATS`](https://github.com/container-storage-interface/spec/blob/master/spec.md#nodegetvolumestats)

Dynamic volume provisioning
---------------------------

`topolvm-node` watches [`LogicalVolume`](./crd-logical-volume.md) and creates
or deletes LVM logical volumes by sending requests to [`lvmd`](./lvmd.md).

### Create a logical volume

If `logicalvolume.status.volumeID` is empty,
it means that the logical volume correspond to `LogicalVolume` is not provisioned with `lvmd`.
So in that case, `topolvm-node` sends `CreateLV` request to `lvmd`.
If its response is succeeded, `topolvm-node` set `logicalvolume.status.volumeID`.

### Finalize LogicalVolume

When a `LogicalVolume` resource is being deleted, `topolvm-node` sends 
a `RemoveLV` request to `lvmd`.

Prometheus metrics
------------------

### `topolvm_volumegroup_available_bytes`

`topolvm_volumegroup_available_bytes` is a Gauge that indicates the available
free space in the LVM volume group in bytes.

| Label  | Description            |
| ------ | ---------------------- |
| `node` | The node resource name |

Node resource
-------------

`topolvm-node` adds `topolvm.cybozu.com/capacity` annotation to the
corresponding `Node` resource of the running node.  The value is the
free storage capacity reported by `lvmd` in bytes.

It also adds `topolvm.cybozu.com/node` finalizer to the `Node`.
The finalizer will be processed by [`topolvm-controller`](./topolvm-controller.md)
to clean up PVCs and associated Pods bound to the node.

Command-line flags
------------------

| Name           | Type   | Default                         | Description                            |
| -------------- | ------ | ------------------------------- | -------------------------------------- |
| `csi-socket`   | string | `/run/topolvm/csi-topolvm.sock` | UNIX domain socket of `topolvm-node`.  |
| `lvmd-socket`  | string | `/run/topolvm/lvmd.sock`        | UNIX domain socket of `lvmd` service.  |
| `metrics-addr` | string | `:8080`                         | Bind address for the metrics endpoint. |
| `nodename`     | string |                                 | `Node` resource name.                  |

Environment variables
---------------------

- `NODE_NAME`: `Node` resource name.

If both `NODE_NAME` and `nodename` flag are given, `nodename` flag is preceded.
