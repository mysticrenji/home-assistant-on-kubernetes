# k3s-on-nvidia-jetson
K3s Cluster on Nvidia Jetson

## K3s Install on Nvidia Jetson ARM64 utlizing Containerd Runtime
```
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --disable servicelb --disable traefik --write-kubeconfig-mode 644 --cluster-cidr=10.10.0.0/16" INSTALL_K3S_VERSION="v1.24.8+k3s1" sh -s -
```

## Install K8s Device Plugin for Nvidia
```
kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v0.10.0/nvidia-device-plugin.yml
```

## Install MetalLB
```
Instll MetalLB
--------------
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/metallb.yaml

# Create a secret for encrypted speaker communications
$ kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"

Config for metallb
------------------
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: address-pool-1
      protocol: layer2
      addresses:
      - 192.168.2.128/25

```

## Install Cert Manager
```
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set installCRDs=true
```

