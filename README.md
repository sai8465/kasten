## installing k3s cluster

```
curl -sfL https://get.k3s.io | VolumeSnapshotDataSource=true sh -s
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
sudo chmod 644 $KUBECONFIG
```

### Installing Kasten

# Pre-Flight Checks

Run the following command to deploy the the pre-check tool:

```
curl https://docs.kasten.io/tools/k10_primer.sh | bash
```


# Prerequisites 

Add the Kasten Helm charts repository using:

```
helm repo add kasten https://charts.kasten.io/
```

You need to create the namespace where Kasten will be installed. By default, the documentation uses kasten-io.

```
kubectl create ns kasten-io
```
# Installing k10

To install K10 on k3s, you also need to annotate the default VolumeSnapshotClass as specified in our CSI documentation.

```
helm install k10 kasten/k10 --namespace=kasten-io
kubectl annotate volumesnapshotclass \
$(kubectl get volumesnapshotclass -o=jsonpath='{.items[?(@.metadata.annotations.snapshot\.storage\.kubernetes\.io\/is-default-class=="true")].metadata.name}') \
k10.kasten.io/is-snapshot-class=true
```
Install the VolumeSnapshot CRDS and the Snapshot Controller
```
# Install a recent version of the CSI snapshotter
SNAPSHOTTER_VERSION=v6.0.1

# Apply VolumeSnapshot CRDs
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/${SNAPSHOTTER_VERSION}/client/config/crd/snapshot.storage.k8s.io_volumesnapshotclasses.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/${SNAPSHOTTER_VERSION}/client/config/crd/snapshot.storage.k8s.io_volumesnapshotcontents.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/${SNAPSHOTTER_VERSION}/client/config/crd/snapshot.storage.k8s.io_volumesnapshots.yaml

# Create Snapshot Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/${SNAPSHOTTER_VERSION}/deploy/kubernetes/snapshot-controller/rbac-snapshot-controller.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/${SNAPSHOTTER_VERSION}/deploy/kubernetes/snapshot-controller/setup-snapshot-controller.yaml
```
## Install the CSI Hostpath Driver
```
git clone https://github.com/kubernetes-csi/csi-driver-host-path.git
cd csi-driver-host-path
./deploy/kubernetes-latest/deploy.sh
```
```
for i in ./examples/csi-storageclass.yaml ./examples/csi-pvc.yaml ./examples/csi-app.yaml; do kubectl apply -f $i; done
```
```
kubectl apply -f ./examples/csi-storageclass.yaml
kubectl patch storageclass standard \
    -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
kubectl patch storageclass csi-hostpath-sc \
    -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```    

To validate that K10 has been installed properly.

```
kubectl get pods --namespace kasten-io --watch
```
By default, the K10 dashboard will not be exposed externally. To establish a connection to it, use the following kubectl command to forward a local port to the K10 ingress port:

```
kubectl --namespace kasten-io port-forward service/gateway 8080:8000
```
The K10 dashboard will be available at http://127.0.0.1:8080/k10/#/

## dashboard
![image](https://user-images.githubusercontent.com/79599130/181222950-7dae3288-c258-4b4a-a80b-bb94675b68e9.png)

## Settings

![image](https://user-images.githubusercontent.com/79599130/181240819-d35b0b7a-eaa3-4830-b8e2-91ab6dea9973.png)

# Location Configuration

K10 can usually invoke protection operations such as snapshots within a cluster without requiring additional credentials. While this might be sufficient if K10 is running in some of (but not all) the major public clouds and if actions are limited to a single cluster, it is not sufficient for essential operations such as performing real backups, enabling cross-cluster and cross-cloud application migration, and enabling DR of the K10 system itself.


![image](https://user-images.githubusercontent.com/79599130/181241622-ab3fcdb8-5181-442a-a48c-a10d853985b2.png)

## Location Profiles
Location profiles are used to create backups from snapshots, move applications and their data across clusters and potentially across different clouds, and to subsequently import these backups or exports into another cluster. To create a location profile, click New Profile on the profiles page.

### Object Storage Location
Support is available for the following object storage providers:

- Amazon S3 or S3 Compatible Storage
- Azure Storage
- Google Cloud Storage

K10 creates Kopia repositories in object store locations. K10 uses Kopia as a data mover which implicitly provides support to deduplicate, encrypt and compress data at rest. K10 performs periodic maintenance on these repositories to recover released storage.

## Amazon S3 or S3 Compatible Storage

![image](https://user-images.githubusercontent.com/79599130/181243777-bd87941b-f019-4922-8c99-fe10e40326a7.png)

The config profile will be created and a profile similar to the following will appear:

![image](https://user-images.githubusercontent.com/79599130/181243934-55669edc-3d79-4f64-91b9-076db5152599.png)

## Creating  backup policies

![image](https://user-images.githubusercontent.com/79599130/181247040-d62cc743-f07c-47d7-a354-e47ce6b746d8.png)

select bucket 

![image](https://user-images.githubusercontent.com/79599130/181247135-aca42495-dcd5-43a5-87cc-a1496efc8cc8.png)

Enable Snapshot Cluster-Scoped Resources

![image](https://user-images.githubusercontent.com/79599130/181247431-332828e8-3ff6-48b8-b7e4-589492be4246.png)

click create policy

![image](https://user-images.githubusercontent.com/79599130/181248562-a2e73791-b72e-4d9c-9da5-936457de507b.png)


Here automatically genarate config like 

![image](https://user-images.githubusercontent.com/79599130/181249330-223f6c35-3b4c-4040-9d6e-f2a76469e647.png)


# Restoring data on other cluster 

Creating restore policy 

![image](https://user-images.githubusercontent.com/79599130/181249596-e60bd083-77ae-49db-a64e-0cbc89cd7dbe.png)

![image](https://user-images.githubusercontent.com/79599130/181249657-5be6a5aa-f879-413c-a471-604d1141230a.png)

![image](https://user-images.githubusercontent.com/79599130/181249684-74cbce48-756b-4294-9a5f-618300b9c373.png)

![image](https://user-images.githubusercontent.com/79599130/181249748-b9b353b3-e023-4f17-84d0-bb820bcde24d.png)

![image](https://user-images.githubusercontent.com/79599130/181249814-8a0369f5-9454-4cb9-99a4-ded2622a3c42.png)

![image](https://user-images.githubusercontent.com/79599130/181249861-87fa7a87-b67a-4b8d-9c17-814f5968cb82.png)
