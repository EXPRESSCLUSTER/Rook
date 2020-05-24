# Rook-Ceph
- [Evaluation Environment](#evaluation-environment)
- [Prerequisite](#prerequisite)
- [Deploy Ceph](#deploy-ceph)

## Evaluation Environment
- Each worker node of kubernetes has 20GB disk (e.g. /dev/sdb) for Ceph.
  ```
  +-------------------------------------------+
  | CentOS Linux release 7.8.2003 (Core)      | 
  | - KVM                                     |
  |   Compiled against library: libvirt 4.5.0 |
  |   Using library: libvirt 4.5.0            |
  |   Using API: QEMU 4.5.0                   |
  |   Running hypervisor: QEMU 2.12.0         |
  | +------------------------+                |
  | | Master (Control Plane) |                |
  | | Ubuntu 18.04.4 LTS     |                |
  | | kubernetes 1.18.3      |                |
  | | Docker 19.03.9         |                |
  | +------+-----------------+                |
  |        |                                  |
  | +------+-------------+  +----------+      |
  | | Worker #1, #2, #3  |  |          |      |
  | | Ubuntu 18.04.4 LTS +--+ /dev/sdb |      |
  | | kubernetes 1.18.3  |  |     20GB |      |
  | | Docker 19.03.9     |  |          |      |
  | +--------------------+  +----------+      |
  +-------------------------------------------+
  ```
## Prerequisite
- Install git command on master node.

## Deploy Ceph
1. Create a kubernetes cluster with 1 master and 3 workers refer to the following page.
   - https://github.com/EXPRESSCLUSTER/kubernetes/blob/master/HowToInstallK8s.md
1. Download the files from Rook GitHub.
   ```sh
   $ git clone https://github.com/rook/rook.git
   ```
1. Apply the manifest files.
   ```sh
   $ kubectl apply -f cluster/example/kubernetes/ceph/common.yaml
   $ kubectl apply -f cluster/example/kubernetes/ceph/operator.yaml
   ```
1. Edit the cluster.yaml as below.
   ```yaml
     storage: # cluster level storage configuration and selection
       useAllNodes: true
       useAllDevices: true
       #deviceFilter:
       config:
       nodes:
       - name: "ubuntu18-52"
         devices:
         - name: "sdb"
       - name: "ubuntu18-53"
         devices:
         - name: "sdb"
       - name: "ubuntu18-54"
         devices:
         - name: "sdb"
   ```
1. Apply the manifest file.
   ```sh
   $ kubectl apply -f cluster/example/kubernetes/ceph/cluster.yaml
   ```
1. Edit the ConfigMap.
   ```sh
   $ kubectl -n rook-ceph edit configmap rook-config-override
   ```
1. If clock skew erorr occurs on mon pod, set the following parameter and reboot the worker nodes.
   ```yaml
   apiVersion: v1
   data:
     config: |
       [mon]
         mon clock drift allowed = 5.0
   kind: ConfigMap
    :
   ```
   - For more details of clock skew, refer to the following page.
     - https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/4/html-single/troubleshooting_guide/index#clock-skew_diag
     - https://rook.io/docs/rook/v0.8/advanced-configuration.html#custom-cephconf-settings
     - https://docs.ceph.com/docs/jewel/rados/configuration/mon-config-ref/
1. Check if the following pods' status are running or completed.
   ```sh
   $kubectl get -n rook-ceph pod
   NAME                                                    READY   STATUS      RESTARTS   AGE
   csi-cephfsplugin-7bbxn                                  3/3     Running     3          8h
   csi-cephfsplugin-m2k8s                                  3/3     Running     3          8h
   csi-cephfsplugin-provisioner-7459667b67-gmvr9           5/5     Running     6          8h
   csi-cephfsplugin-provisioner-7459667b67-vtxlm           5/5     Running     6          8h
   csi-cephfsplugin-qpm26                                  3/3     Running     3          8h
   csi-rbdplugin-kv48r                                     3/3     Running     3          8h
   csi-rbdplugin-provisioner-7cbb778994-4q2g7              6/6     Running     12         8h
   csi-rbdplugin-provisioner-7cbb778994-dhwst              6/6     Running     7          8h
   csi-rbdplugin-rzpp5                                     3/3     Running     4          8h
   csi-rbdplugin-vz4rh                                     3/3     Running     3          8h
   rook-ceph-crashcollector-ubuntu18-52-694bf6b456-twmvd   1/1     Running     1          5h54m
   rook-ceph-crashcollector-ubuntu18-53-644ff69867-s9t7f   1/1     Running     1          5h13m
   rook-ceph-crashcollector-ubuntu18-54-778f9746fd-b6tjg   1/1     Running     1          5h55m
   rook-ceph-mgr-a-64fd689dc8-7mw5j                        1/1     Running     0          5h6m
   rook-ceph-mon-a-77d6f876bd-l8w5l                        1/1     Running     1          5h55m
   rook-ceph-mon-b-64f6c5fb45-4rwn7                        1/1     Running     1          5h54m
   rook-ceph-mon-c-9b4bc6866-528tg                         1/1     Running     1          5h54m
   rook-ceph-operator-567d7945d6-rvkft                     1/1     Running     1          6h9m
   rook-ceph-osd-0-85c8c8758-ll7vp                         1/1     Running     1          5h13m
   rook-ceph-osd-1-57d5cf4fd5-flvlk                        1/1     Running     1          5h13m
   rook-ceph-osd-2-94c6db8d4-vzxwc                         1/1     Running     0          4h58m
   rook-ceph-osd-prepare-ubuntu18-52-xss7z                 0/1     Completed   0          5h
   rook-ceph-osd-prepare-ubuntu18-53-djmnt                 0/1     Completed   0          5h
   rook-ceph-osd-prepare-ubuntu18-54-fzl9v                 0/1     Completed   0          5h
   rook-discover-6ss5r                                     1/1     Running     1          8h
   rook-discover-wdcsx                                     1/1     Running     1          8h
   rook-discover-wdwl4                                     1/1     Running     1          8h
   ```
1. Create a test container.
   ```sh
   $ kubectl apply -f cluster/example/kubernetes/ceph/toolbox.yaml
   ```
1. Log in the container and check ceph's status.
   ```sh
   $ kubectl exec -n rook-ceph exec rook-ceph-tools-6d659f5579-v9f2p -- bash
   [root@rook-ceph-tools-6d659f5579-v9f2p /]# ceph -s
     cluster:
       id:     18d395a7-0d39-455a-b091-28801667f660
       health: HEALTH_OK
   
     services:
       mon: 3 daemons, quorum a,b,c (age 4h)
       mgr: a(active, since 5h)
       osd: 3 osds: 3 up (since 5h), 3 in (since 5h)
   
     data:
       pools:   0 pools, 0 pgs
       objects: 0 objects, 0 B
       usage:   3.0 GiB used, 57 GiB / 60 GiB avail
       pgs:
   
   [root@rook-ceph-tools-6d659f5579-v9f2p /]# ceph df
   RAW STORAGE:
       CLASS     SIZE       AVAIL      USED        RAW USED     %RAW USED
       hdd       60 GiB     57 GiB     6.9 MiB      3.0 GiB          5.01
       TOTAL     60 GiB     57 GiB     6.9 MiB      3.0 GiB          5.01
   
   POOLS:
       POOL     ID     STORED     OBJECTS     USED     %USED     MAX AVAIL   
   ```
