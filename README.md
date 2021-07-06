# K8s cluster using CRI-O

The ansible playbook is based on [Kubernetes setup with CRI-O Runtime](https://github.com/msfidelis/kubernetes-with-cri-o)

I have also added a storage with iSCSI and NFS share

If you need a kubernetes bare-matal using kvm vm check the [k8s-kvm project](https://github.com/ssorato/ansible-kvm)

See the [inventory file](inventories/lab.yml)

Check if all hosts are availabe:

```bash
$ ansible -i inventories/lab.yml -m ping all
```

Execute the playbook

```bash
$ ansible-playbook -i inventories/lab.yml main.yml
```

The `kubectl` config file is localted in the master node as `/etc/kubernetes/admin.conf`

Checking the cluster's nodes

```bash
$ kubectl get nodes
NAME          STATUS   ROLES                  AGE     VERSION
k8s-master    Ready    control-plane,master   3m43s   v1.21.2
k8s-worker1   Ready    <none>                 3m18s   v1.21.2
k8s-worker2   Ready    <none>                 3m18s   v1.21.2
```

Check the NFS/iSCI server ( execute the commands inside the host )

```bash
$ sudo exportfs -s
/data  192.168.123.0/24(rw,wdelay,no_root_squash,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)

$ sudo iscsiadm -m discovery -t st -p 192.168.123.64
192.168.123.64:3260,1 iqn.2020-07.lab.com:lun1
```