# Rook-Ceph
- [Evaluation Environment](#evaluation-environment)
- [Prerequisite](#prerequisite)
- [Deploy Ceph](#deploy-ceph)

## Evaluation Environment
- Each worker node of kubernetes has 20GB disk (e.g. /dev/sdb) for Ceph.
  ```
  +-------------------------------------------+
  | Windows Server                            | 
  | - Hyper-V                                 |
  | +------------------------+                |
  | | Master (Control Plane) |                |
  | | Ubuntu 20.04 LTS       |                |
  | | kubernetes 1.18.3      |                |
  | | Docker 19.03.10        |                |
  | +------+-----------------+                |
  |        |                                  |
  | +------+-------------+  +----------+      |
  | | Worker #1, #2, #3  |  |          |      |
  | | Ubuntu 20.04   LTS +--+ /dev/sdb |      |
  | | kubernetes 1.18.3  |  |     40GB |      |
  | | Docker 19.03.10    |  |          |      |
  | +--------------------+  +----------+      |
  +-------------------------------------------+
  ```
## Prerequisite
- Install git command on master node.
- Ceph uses Paxos algorithm. If your machines cannot communicate within Ceph's timeout, Ceph cluster does not work well. You need to check Ceph's system requirement.

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
1. Apply the manifest file.
   ```sh
   $ kubectl apply -f cluster/example/kubernetes/ceph/cluster.yaml
   ```
1. Check if the following pods' status are running or completed.
   ```sh
   $kubectl get -n rook-ceph pod
   NAME                                                   READY   STATUS      RESTARTS   AGE
   csi-cephfsplugin-fmmln                                 3/3     Running     0          2d
   csi-cephfsplugin-j5rgf                                 3/3     Running     0          2d
   csi-cephfsplugin-mrd75                                 3/3     Running     0          2d
   csi-cephfsplugin-provisioner-7678bcfc46-g9zk4          5/5     Running     0          2d
   csi-cephfsplugin-provisioner-7678bcfc46-zqht7          5/5     Running     0          2d
   csi-rbdplugin-g7dmk                                    3/3     Running     0          2d
   csi-rbdplugin-lh4b6                                    3/3     Running     1          2d
   csi-rbdplugin-provisioner-fbd45b7c8-blmt6              6/6     Running     0          2d
   csi-rbdplugin-provisioner-fbd45b7c8-xpc5j              6/6     Running     0          2d
   csi-rbdplugin-sv9kg                                    3/3     Running     0          2d
   rook-ceph-crashcollector-ubuntu-206-8f888bcfb-2bwnv    1/1     Running     1          3d1h
   rook-ceph-crashcollector-ubuntu-207-56d9446dd5-4phx5   1/1     Running     1          3d1h
   rook-ceph-crashcollector-ubuntu-208-5485ddfb94-rq62z   1/1     Running     1          3d1h
   rook-ceph-mgr-a-64bdc77cfc-87whl                       1/1     Running     2          3d1h
   rook-ceph-mon-a-6958898686-jz69c                       1/1     Running     1          3d1h
   rook-ceph-mon-b-6798c7fbf6-ld5rb                       1/1     Running     1          3d1h
   rook-ceph-mon-c-5c45b78579-xgfxk                       1/1     Running     1          3d1h
   rook-ceph-operator-757d6db48d-zpnlj                    1/1     Running     0          2d1h
   rook-ceph-osd-0-6b98978859-h688n                       1/1     Running     3          3d1h
   rook-ceph-osd-1-845cfddd85-j6lkw                       1/1     Running     2          3d1h
   rook-ceph-osd-2-c7c4968bd-b5dxh                        1/1     Running     1          3d1h
   rook-ceph-osd-prepare-ubuntu-206-sxrln                 0/1     Completed   0          2d
   rook-ceph-osd-prepare-ubuntu-207-5zqls                 0/1     Completed   0          2d
   rook-ceph-osd-prepare-ubuntu-208-rsz4m                 0/1     Completed   0          2d
   rook-discover-6thpp                                    1/1     Running     0          2d
   rook-discover-8mrmw                                    1/1     Running     0          2d
   rook-discover-mdxmm                                    1/1     Running     0          2d
   ```
