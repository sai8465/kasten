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
To validate that K10 has been installed properly.

```
kubectl get pods --namespace kasten-io --watch
```
By default, the K10 dashboard will not be exposed externally. To establish a connection to it, use the following kubectl command to forward a local port to the K10 ingress port:

```
kubectl --namespace kasten-io port-forward service/gateway 8080:8000
```
The K10 dashboard will be available at http://127.0.0.1:8080/k10/#/

![image](https://user-images.githubusercontent.com/79599130/181217632-1d34f5dc-7a90-417a-9a22-80d08dcbd82a.png)

### settings

![image](https://user-images.githubusercontent.com/79599130/181217769-711b6159-d7a3-47a7-8aa3-701bb3830722.png)

### Ctrating Location

![image](https://user-images.githubusercontent.com/79599130/181217891-edb15db0-1ac4-43fb-94a5-3fc760e583c5.png)





