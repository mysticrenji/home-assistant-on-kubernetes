# Home-Assistant on K3s Cluster
[![Docker Build for Bluez Service](https://github.com/mysticrenji/home-assistant-on-kubernetes/actions/workflows/main.yaml/badge.svg?branch=main)](https://github.com/mysticrenji/home-assistant-on-kubernetes/actions/workflows/main.yaml)
[![Nightly Build test for Home Assistant](https://github.com/mysticrenji/home-assistant-on-kubernetes/actions/workflows/tests.yaml/badge.svg?branch=main)](https://github.com/mysticrenji/home-assistant-on-kubernetes/actions/workflows/tests.yaml)

Personal project to run home-assistant on K3s locally on Nvidia Jetson(ARM64)

A complete walkthrough on the setup is posted on Medium
-  [Part 1](https://renjithvr11.medium.com/running-your-home-assistant-on-kubernetes-part-i-e66fd24ab8f1)
-  [Part 2](https://renjithvr11.medium.com/running-your-home-assistant-on-kubernetes-part-ii-60eb46a73c61)

## Architecture
![Architecture](./images/Home%20Automation.png)


In this setup, a K3s cluster is being run on a single ARM64 NVidia Jetson Nano device, showcasing the power of containerization technology. One of the pods in this cluster is home-assistant, utilizing the official container image. There are other pods running in the same namespace, such as bluez service, which interfaces with Bluetooth hardware, and a zigbee coordinator service, which interfaces with zigbee hardware, both connected to the Nvidia Jetson using USB ports.

The bluez service pod is capable of fetching messages sent by a Plant soil sensor via Bluetooth Low Energy (BLE). Additionally, the zigbee device sensor data is sent to the zigbee coordinator. These devices are easily mapped to Homeassistant using built-in add-ons that are both available and supported.

To make this setup accessible from outside the network, a Cloudflare tunnel service is running inside the cluster. This allows Home-assistant to be reachable from the outside world. To ensure proper name resolution, Cloudflare name server details from Google Domain Servers are injected into the system, making it easy to access via domain names. Overall, this is a powerful and well-connected system, showcasing the potential of containerization technology for running complex setups on single devices.

## Install K3s on Nvidia Jetson ARM64 utlizing Containerd Runtime
```
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --disable servicelb --disable traefik --write-kubeconfig-mode 644 --cluster-cidr=10.10.0.0/16" INSTALL_K3S_VERSION="v1.31.4+k3s1" sh -s -
```

## Install Kubernetes Device Plugin for Nvidia

For tapping GPU capabilities in K3s cluster, it needs to follow a few steps

[K3s Nvidia Setup](https://docs.k3s.io/advanced#nvidia-container-runtime-support)

```
kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v0.10.0/nvidia-device-plugin.yml
```

## Install Cloudflare Daemon for Secure Public Access

The below will create a deployment with running cloudflare daemon to connect t to cloudflare platform, allowing access to the your internally hosted pods

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
## Add Kubernetes Worker nodes from different network(different location)

1. First we need to make sure that both the control plane and worker nodes are able to communicate with each other. To solve this issue, I have made use of tailscale VPN which will install agents on the ARM64 devices. Tailscale features free connectivity upto 3 nodes
```
curl -fsSL https://tailscale.com/install.sh | sh
```
2. In the worker nodes, use the below code snippet
```
TOKEN="get token from the control server"
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="agent" INSTALL_K3S_VERSION="v1.31.4+k3s1" K3S_URL=http://[ipfromtailscale]:6443 K3S_TOKEN=$TOKEN sh -s - --snapshotter=native
```

## FAQ
1. How to enable HACS Addon on Home Assistant?
   You need to manualy install HACS on the Home assistant. This can be done by spinning up a bash shell inside the Home Assistant Pod and running the below command. Once installed, restart the deployment to see the HACS enabled on the left-hand side panel.
   ```
   wget -O - https://get.hacs.xyz | bash -
   ```
   
## Star History

<a href="https://star-history.com/#mysticrenji/home-assistant-on-kubernetes&Date">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=mysticrenji/home-assistant-on-kubernetes&type=Date&theme=dark" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=mysticrenji/home-assistant-on-kubernetes&type=Date" />
   <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=mysticrenji/home-assistant-on-kubernetes&type=Date" />
 </picture>
</a>
