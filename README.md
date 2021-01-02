# kubeinst

kubeinst is an Ansible playbook, that deploys Kubernetes master/worker nodes. The playboook adds repository for Docker and Kubernetes (Ubuntu/Centos), installs them, initializes Kubernetes master node with Flannel pod network, metrics-server, Kubernetes dashboard, Helm, nfs-server-provisioner, kubecolor. Tested on Ubuntu/Centos, can run on Debian/Redhat too.<br/>
For nfs-server-provisioner Nfs server is deployed on master node by default or you can specify it address and export directory in defaults/main.yaml:
```console
nfs_server_addr: master
nfs_server_export: /var/nfsroot
```
nfs-server-provisioner volume size can be specified in defaults/main.yaml:
```console
helm_inst_nfs: helm install nfs-server-provisioner kube/nfs-server-provisioner --set storageClass.defaultClass=true,persistence.size=1Gi
```

## Extra variables
Extra variables can be passed to playbook:
- worker - specify a worker nodes

## How to run
To deploy master node run
```console
# ANSIBLE_DISPLAY_SKIPPED_HOSTS=no ansible-playbook -i vm-centos2, -u root --key-file ~/.ssh/id_rsa.vm kubeinst.yml
```
To deploy worker node run
```console
# ANSIBLE_DISPLAY_SKIPPED_HOSTS=no ansible-playbook -i vm-centos, -u root --key-file ~/.ssh/id_rsa.vm --extra-vars 'worker=true' kubeinst.yml
```

After deployment of master and worker nodes it will look like about that
```console
root @ vm-centos2 : ~   2020-11-01 15:35:15    %Cpu(s):6.7us 6.7sy 0.0ni 86.7id 0.0wa 0.0hi 0.0si 0.0st 
# kubectl get all,pv,pvc,node -A -owide
NAMESPACE              NAME                                             READY   STATUS    RESTARTS   AGE   IP              NODE               NOMINATED NODE   READINESS GATES
default                pod/nfs-server-provisioner-0                     1/1     Running   2          25h   10.244.0.22     vm-centos2.local   <none>           <none>
kube-system            pod/coredns-f9fd979d6-gm9gq                      1/1     Running   2          25h   10.244.0.20     vm-centos2.local   <none>           <none>
kube-system            pod/coredns-f9fd979d6-w76vr                      1/1     Running   2          25h   10.244.0.21     vm-centos2.local   <none>           <none>
kube-system            pod/etcd-vm-centos2.local                        1/1     Running   2          25h   192.168.122.4   vm-centos2.local   <none>           <none>
kube-system            pod/kube-apiserver-vm-centos2.local              1/1     Running   2          25h   192.168.122.4   vm-centos2.local   <none>           <none>
kube-system            pod/kube-controller-manager-vm-centos2.local     1/1     Running   5          25h   192.168.122.4   vm-centos2.local   <none>           <none>
kube-system            pod/kube-flannel-ds-9gjt6                        1/1     Running   2          25h   192.168.122.4   vm-centos2.local   <none>           <none>
kube-system            pod/kube-proxy-9n4kk                             1/1     Running   2          25h   192.168.122.4   vm-centos2.local   <none>           <none>
kube-system            pod/kube-scheduler-vm-centos2.local              1/1     Running   5          25h   192.168.122.4   vm-centos2.local   <none>           <none>
kube-system            pod/metrics-server-594c4964f6-lbbrg              1/1     Running   2          25h   10.244.0.19     vm-centos2.local   <none>           <none>
kubernetes-dashboard   pod/dashboard-metrics-scraper-7b59f7d4df-vpxvq   1/1     Running   2          25h   10.244.0.23     vm-centos2.local   <none>           <none>
kubernetes-dashboard   pod/kubernetes-dashboard-6556b6bfdd-gls82        1/1     Running   2          25h   10.244.0.24     vm-centos2.local   <none>           <none>

NAMESPACE              NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                                                     AGE   SELECTOR
default                service/kubernetes                  ClusterIP   10.96.0.1       <none>        443/TCP                                                                                                     25h   <none>
default                service/nfs-server-provisioner      ClusterIP   10.111.76.169   <none>        2049/TCP,2049/UDP,32803/TCP,32803/UDP,20048/TCP,20048/UDP,875/TCP,875/UDP,111/TCP,111/UDP,662/TCP,662/UDP   25h   app=nfs-server-provisioner,release=nfs-server-provisioner
kube-system            service/kube-dns                    ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP                                                                                      25h   k8s-app=kube-dns
kube-system            service/metrics-server              ClusterIP   10.97.6.80      <none>        443/TCP                                                                                                     25h   k8s-app=metrics-server
kubernetes-dashboard   service/dashboard-metrics-scraper   ClusterIP   10.99.188.50    <none>        8000/TCP                                                                                                    25h   k8s-app=dashboard-metrics-scraper
kubernetes-dashboard   service/kubernetes-dashboard        ClusterIP   10.108.125.2    <none>        443/TCP                                                                                                     25h   k8s-app=kubernetes-dashboard

NAMESPACE     NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE   CONTAINERS     IMAGES                           SELECTOR
kube-system   daemonset.apps/kube-flannel-ds   1         1         1       1            1           <none>                   25h   kube-flannel   quay.io/coreos/flannel:v0.13.0   app=flannel
kube-system   daemonset.apps/kube-proxy        1         1         1       1            1           kubernetes.io/os=linux   25h   kube-proxy     k8s.gcr.io/kube-proxy:v1.19.3    k8s-app=kube-proxy

NAMESPACE              NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS                  IMAGES                                            SELECTOR
kube-system            deployment.apps/coredns                     2/2     2            2           25h   coredns                     k8s.gcr.io/coredns:1.7.0                          k8s-app=kube-dns
kube-system            deployment.apps/metrics-server              1/1     1            1           25h   metrics-server              k8s.gcr.io/metrics-server/metrics-server:v0.3.7   k8s-app=metrics-server
kubernetes-dashboard   deployment.apps/dashboard-metrics-scraper   1/1     1            1           25h   dashboard-metrics-scraper   kubernetesui/metrics-scraper:v1.0.4               k8s-app=dashboard-metrics-scraper
kubernetes-dashboard   deployment.apps/kubernetes-dashboard        1/1     1            1           25h   kubernetes-dashboard        kubernetesui/dashboard:v2.0.0                     k8s-app=kubernetes-dashboard

NAMESPACE              NAME                                                   DESIRED   CURRENT   READY   AGE   CONTAINERS                  IMAGES                                            SELECTOR
kube-system            replicaset.apps/coredns-f9fd979d6                      2         2         2       25h   coredns                     k8s.gcr.io/coredns:1.7.0                          k8s-app=kube-dns,pod-template-hash=f9fd979d6
kube-system            replicaset.apps/metrics-server-594c4964f6              1         1         1       25h   metrics-server              k8s.gcr.io/metrics-server/metrics-server:v0.3.7   k8s-app=metrics-server,pod-template-hash=594c4964f6
kubernetes-dashboard   replicaset.apps/dashboard-metrics-scraper-7b59f7d4df   1         1         1       25h   dashboard-metrics-scraper   kubernetesui/metrics-scraper:v1.0.4               k8s-app=dashboard-metrics-scraper,pod-template-hash=7b59f7d4df
kubernetes-dashboard   replicaset.apps/kubernetes-dashboard-6556b6bfdd        1         1         1       25h   kubernetes-dashboard        kubernetesui/dashboard:v2.0.0                     k8s-app=kubernetes-dashboard,pod-template-hash=6556b6bfdd
kubernetes-dashboard   replicaset.apps/kubernetes-dashboard-74d688b6bc        0         0         0       25h   kubernetes-dashboard        kubernetesui/dashboard:v2.0.0                     k8s-app=kubernetes-dashboard,pod-template-hash=74d688b6bc

NAMESPACE   NAME                                      READY   AGE   CONTAINERS               IMAGES
default     statefulset.apps/nfs-server-provisioner   1/1     25h   nfs-server-provisioner   quay.io/kubernetes_incubator/nfs-provisioner:v2.3.0

NAMESPACE   NAME                    STATUS   ROLES    AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION                CONTAINER-RUNTIME
            node/vm-centos.local    Ready    <none>   14h   v1.19.3   192.168.122.3   <none>        CentOS Linux 7 (Core)   3.10.0-1127.19.1.el7.x86_64   docker://19.3.13
            node/vm-centos2.local   Ready    master   21h   v1.19.3   192.168.122.4   <none>        CentOS Linux 7 (Core)   3.10.0-1127.19.1.el7.x86_64   docker://19.3.13

```
