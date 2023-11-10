## CKA notes

ETCD commands for Docker Desktop:

```
kubectl exec etcd-docker-desktop -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --endpoints 127.0.0.1:2379 --cert=/run/config/pki/etcd/server.crt --key=/run/config/pki/etcd/server.key --cacert=/run/config/pki/etcd/ca.crt"
```
