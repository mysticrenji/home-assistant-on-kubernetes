# Home-Assistant on K3s Cluster
Personal project to run home-assistant on K3s locally on Nvidia Jetson(ARM64)

## Architecture
![Architecture](./images/Home%20Automation.png)

## K3s Install on Nvidia Jetson ARM64 utlizing Containerd Runtime
```
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --disable servicelb --disable traefik --write-kubeconfig-mode 644 --cluster-cidr=10.10.0.0/16" INSTALL_K3S_VERSION="v1.26.3+k3s1" sh -s -
```

## Install Kubernetes Device Plugin for Nvidia

For tapping GPU capabilities in K3s cluster, it needs to follow a few steps

[K3s Nvidia Setup](https://docs.k3s.io/advanced#nvidia-container-runtime-support)

```
kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v0.10.0/nvidia-device-plugin.yml
```

## Install Cloudflare Daemon for Secure Public Access

The below will create a deployment with running cloudflare daemon to connect to connect to cloudflare platform, allowing access to the your internally hosted pods

```
kubectl create -f ./cloudflare-daemon/deployment.yaml
```

## Installing MetalLB 
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

