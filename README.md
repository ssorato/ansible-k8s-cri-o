# K8s cluster using CRI-O

The ansible playbook is based on [Kubernetes setup with CRI-O Runtime](https://github.com/msfidelis/kubernetes-with-cri-o)

I have also added a storage with iSCSI and NFS share and a MetalLB load balancer.

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

$ sudo iscsiadm -m discovery -t st -p 192.168.123.63
192.168.123.64:3260,1 iqn.2020-07.lab.com:lun1
```

# Traefik Ingress Controller demo application

Install Traefik Ingress

```bash
$ helm repo add traefik https://helm.traefik.io/traefik
$ helm repo update
$ helm install traefik traefik/traefik
```

deploy the application

```bash
$ kubectl apply -f demo-traefik/
```

Get the load balancer IP

```bash
$ kubectl get svc
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)                      AGE
kubernetes   ClusterIP      10.96.0.1       <none>            443/TCP                      82m
traefik      LoadBalancer   10.96.144.154   192.168.123.250   80:31272/TCP,443:30990/TCP   4m1s
whoami       ClusterIP      10.110.70.160   <none>            80/TCP                       5m19s
```

Test the application

```bash
$ curl 192.168.123.250
```

# Nginx Ingress Controller demo application

Install Nginx Ingress

```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.0/deploy/static/provider/baremetal/deploy.yaml
```

deploy the application

```bash
$ kubectl apply -f demo-nginx/
```

Get the load balancer IP

```bash
$ kubectl -n ingress-nginx get svc ingress-nginx-controller
NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)                      AGE
ingress-nginx-controller   LoadBalancer   10.99.148.104   192.168.123.250   80:31644/TCP,443:30111/TCP   6m31s
```

Test the application

```bash
$ curl http://192.168.123.250/apple
apple

$ curl http://192.168.123.250/banana
banana
```