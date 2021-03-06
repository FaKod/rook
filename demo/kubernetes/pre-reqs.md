
## New Local Kubernetes Cluster
For a quick start with a new local cluster, use the Rook fork of [coreos-kubernetes](https://github.com/rook/coreos-kubernetes). This will bring up a multi-node Kubernetes cluster with `vagrant` and CoreOS virtual machines ready to use `rook` immediately.
```
git clone https://github.com/rook/coreos-kubernetes.git
cd coreos-kubernetes/multi-node/vagrant
vagrant up
export KUBECONFIG="$(pwd)/kubeconfig"
kubectl config use-context vagrant-multi
```

Then wait for the cluster to come up and verify that kubernetes is done initializing (be patient, it takes a bit):

```
kubectl cluster-info
```

Once you see a url response, your cluster is [ready for use by Rook](README.md#deploy-rook).

## Existing Kubernetes Cluster
Alternatively, if you already have a running Kubernetes cluster, you can deploy Rook to it with a small update to modify the kubelet service to bind mount `/sbin/modprobe`, which allows access to `modprobe`.
Access to modprobe is necessary for using the rbd volume plugin, which is being tracked in the Kubernetes code base with [#23924](https://github.com/kubernetes/kubernetes/issues/23924).  

If using RKT, you can enable `modprobe` by following this [guide](https://github.com/coreos/coreos-kubernetes/blob/master/Documentation/kubelet-wrapper.md#allow-pods-to-use-rbd-volumes). Instructions have been directly copied below for your convenience:  

Add the following options to the `RKT_OPTS` env before launching the kubelet via kubelet-wrapper:
```ini
[Service]
Environment=KUBELET_VERSION=v1.5.3_coreos.0
Environment="RKT_OPTS=--volume modprobe,kind=host,source=/usr/sbin/modprobe \
  --mount volume=modprobe,target=/usr/sbin/modprobe \
  --volume lib-modules,kind=host,source=/lib/modules \
  --mount volume=lib-modules,target=/lib/modules \
  --uuid-file-save=/var/run/kubelet-pod.uuid"
...
```

Note that the kubelet also requires access to the userspace `rbd` tool that is included only in hyperkube images tagged `v1.3.6_coreos.0` or later.

## Using Rook in Kubernetes
Now that you have a Kubernetes cluster running, you can start using `rook` with [these steps](README.md#deploy-rook).