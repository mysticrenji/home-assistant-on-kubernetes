# Home Assistant Helm Chart

A Helm chart for deploying [Home Assistant](https://www.home-assistant.io/) on Kubernetes with optional Z-Wave JS and Mosquitto MQTT broker sidecars.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.x
- A PersistentVolume provisioner (e.g. local-path, longhorn, NFS) for config storage
- [cert-manager](https://cert-manager.io/) installed in the cluster (only if `certIssuer.enabled: true`)
- [ingress-nginx](https://kubernetes.github.io/ingress-nginx/) or another Ingress controller (only if `ingress.enabled: true`)

## Quick Start

```bash
# Add the Helm repository
helm repo add home-assistant https://mysticrenji.github.io/home-assistant-on-kubernetes
helm repo update

# Install with default values
helm install my-ha home-assistant/home-assistant

# Or install with a custom values file
helm install my-ha home-assistant/home-assistant -f my-values.yaml
```

### Install from local source (for development)

```bash
git clone https://github.com/mysticrenji/home-assistant-on-kubernetes.git
cd home-assistant-on-kubernetes
helm install my-ha ./charts/home-assistant -f my-values.yaml
```

## What Gets Deployed

| Component | Description | Default |
|-----------|-------------|---------|
| **Home Assistant** | Core home automation platform (port 8123) | Always deployed |
| **Bluez sidecar** | Bluetooth/BLE support for sensors like plant soil monitors | Enabled |
| **Matter Server sidecar** | Matter/Thread smart home protocol support (port 5580) | Enabled |
| **Z-Wave JS** | Z-Wave device management UI and WebSocket server (ports 8091, 3000) | Enabled |
| **Mosquitto** | MQTT message broker for device communication (ports 1883, 9001) | Enabled |
| **Ingress** | NGINX Ingress with optional TLS via cert-manager | Enabled |
| **ClusterIssuer** | Let's Encrypt certificate issuer for TLS | Enabled |

## Configuration Reference

### Global

| Parameter | Description | Default |
|-----------|-------------|---------|
| `namespace` | Kubernetes namespace for all resources | `home-assistant` |

### Home Assistant

| Parameter | Description | Default |
|-----------|-------------|---------|
| `homeAssistant.image.repository` | Container image | `homeassistant/home-assistant` |
| `homeAssistant.image.tag` | Image tag | `2026.2.1` |
| `homeAssistant.resources.requests.memory` | Memory request | `256Mi` |
| `homeAssistant.resources.limits.memory` | Memory limit | `512Mi` |
| `homeAssistant.pvc.size` | Persistent volume size for `/config` | `9Gi` |
| `homeAssistant.pvc.storageClass` | StorageClass name (empty = cluster default) | `""` |
| `homeAssistant.hostNetwork` | Use host networking (required for mDNS/device discovery) | `true` |
| `homeAssistant.privileged` | Run in privileged mode (required for USB/Bluetooth access) | `true` |
| `homeAssistant.configuration` | Contents of `configuration.yaml` injected via ConfigMap | *(see values.yaml)* |

### Home Assistant Devices

| Parameter | Description | Default |
|-----------|-------------|---------|
| `homeAssistant.devices.zigbee.enabled` | Mount Zigbee USB dongle into the pod | `false` |
| `homeAssistant.devices.zigbee.path` | Host path to Zigbee device | `/dev/ttyUSB1` |
| `homeAssistant.devices.dbus.enabled` | Mount D-Bus socket (needed for Bluetooth) | `true` |
| `homeAssistant.devices.dbus.path` | Host path to D-Bus socket | `/run/dbus` |

### Bluez Sidecar (Bluetooth)

| Parameter | Description | Default |
|-----------|-------------|---------|
| `homeAssistant.bluez.enabled` | Deploy Bluez sidecar for BLE support | `true` |
| `homeAssistant.bluez.image.repository` | Bluez image | `ghcr.io/mysticrenji/bluez-service` |
| `homeAssistant.bluez.image.tag` | Bluez image tag | `v1.0.0` |

### Matter Server Sidecar

| Parameter | Description | Default |
|-----------|-------------|---------|
| `homeAssistant.matter.enabled` | Deploy Matter Server sidecar | `true` |
| `homeAssistant.matter.image.repository` | Matter Server image | `ghcr.io/home-assistant-libs/python-matter-server` |
| `homeAssistant.matter.image.tag` | Matter Server image tag | `stable` |
| `homeAssistant.matter.port` | Matter Server port | `5580` |

### Z-Wave JS

| Parameter | Description | Default |
|-----------|-------------|---------|
| `zwavejs.enabled` | Deploy Z-Wave JS | `true` |
| `zwavejs.image.repository` | Container image | `zwavejs/zwavejs2mqtt` |
| `zwavejs.image.tag` | Image tag | `11.11.0` |
| `zwavejs.resources.requests.memory` | Memory request | `64Mi` |
| `zwavejs.resources.requests.cpu` | CPU request | `50m` |
| `zwavejs.resources.limits.memory` | Memory limit | `256Mi` |
| `zwavejs.resources.limits.cpu` | CPU limit | `500m` |
| `zwavejs.pvc.size` | Persistent volume size | `500Mi` |
| `zwavejs.pvc.storageClass` | StorageClass name (empty = cluster default) | `""` |
| `zwavejs.service.type` | Service type (`NodePort`, `ClusterIP`, `LoadBalancer`) | `NodePort` |
| `zwavejs.service.nodePort` | NodePort number (only when type is `NodePort`) | `30091` |
| `zwavejs.devices.zwave.path` | Host path to Z-Wave USB dongle (empty = skip mount) | `/dev/ttyACM0` |

### Mosquitto (MQTT)

| Parameter | Description | Default |
|-----------|-------------|---------|
| `mosquitto.enabled` | Deploy Mosquitto MQTT broker | `true` |
| `mosquitto.image.repository` | Container image | `eclipse-mosquitto` |
| `mosquitto.image.tag` | Image tag | `latest` |
| `mosquitto.resources.requests.memory` | Memory request | `32Mi` |
| `mosquitto.resources.requests.cpu` | CPU request | `25m` |
| `mosquitto.resources.limits.memory` | Memory limit | `128Mi` |
| `mosquitto.resources.limits.cpu` | CPU limit | `250m` |
| `mosquitto.service.type` | Service type | `ClusterIP` |
| `mosquitto.ports.mqtt` | MQTT port | `1883` |
| `mosquitto.ports.websocket` | WebSocket port | `9001` |

### Ingress

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ingress.enabled` | Create an Ingress resource | `true` |
| `ingress.className` | Ingress class name | `nginx` |
| `ingress.host` | Hostname for the Ingress rule | `home-automation.io` |
| `ingress.tls.enabled` | Enable TLS on the Ingress | `true` |
| `ingress.tls.secretName` | TLS Secret name (created by cert-manager or manually) | `letsencrypt` |

### Certificate Issuer

| Parameter | Description | Default |
|-----------|-------------|---------|
| `certIssuer.enabled` | Create a Let's Encrypt ClusterIssuer | `true` |
| `certIssuer.email` | Email for Let's Encrypt registration | `""` |
| `certIssuer.server` | ACME server URL | `https://acme-v02.api.letsencrypt.org/directory` |

## Example Configurations

### Minimal install (no hardware, no ingress)

Suitable for testing or cloud VMs with no USB devices attached:

```yaml
# minimal-values.yaml
homeAssistant:
  hostNetwork: false
  privileged: false
  bluez:
    enabled: false
  matter:
    enabled: false
  devices:
    zigbee:
      enabled: false
    dbus:
      enabled: false

zwavejs:
  enabled: false

mosquitto:
  enabled: true

ingress:
  enabled: false

certIssuer:
  enabled: false
```

```bash
helm install my-ha home-assistant/home-assistant -f minimal-values.yaml
```

### Full smart home setup with Zigbee + Z-Wave + Bluetooth

For a device like Nvidia Jetson or Raspberry Pi with USB dongles:

```yaml
# full-values.yaml
namespace: home-assistant

homeAssistant:
  pvc:
    size: 20Gi
    storageClass: local-path
  devices:
    zigbee:
      enabled: true
      path: /dev/ttyUSB1    # Your Zigbee dongle path (check with: ls /dev/tty*)
    dbus:
      enabled: true
  bluez:
    enabled: true
  matter:
    enabled: true
  configuration: |
    default_config:
    frontend:
      themes: !include_dir_merge_named themes
    tts:
      - platform: google_translate
    automation: !include automations.yaml
    script: !include scripts.yaml
    scene: !include scenes.yaml
    http:
      use_x_forwarded_for: true
      trusted_proxies:
        - 10.10.0.0/16

zwavejs:
  enabled: true
  pvc:
    size: 1Gi
  devices:
    zwave:
      path: /dev/ttyACM0   # Your Z-Wave dongle path

mosquitto:
  enabled: true

ingress:
  enabled: true
  host: home.example.com
  tls:
    enabled: true
    secretName: home-tls

certIssuer:
  enabled: true
  email: you@example.com
```

```bash
helm install my-ha home-assistant/home-assistant -f full-values.yaml
```

### Custom Home Assistant configuration.yaml

The `homeAssistant.configuration` value is injected as `/config/configuration.yaml` inside the pod. You can customize it with any valid Home Assistant YAML:

```yaml
homeAssistant:
  configuration: |
    default_config:
    frontend:
      themes: !include_dir_merge_named themes
    tts:
      - platform: google_translate
    automation: !include automations.yaml
    script: !include scripts.yaml
    scene: !include scenes.yaml
    http:
      use_x_forwarded_for: true
      trusted_proxies:
        - 10.0.0.0/8
    homeassistant:
      auth_providers:
        - type: trusted_networks
          trusted_networks:
            - 10.0.0.0/16
            - 127.0.0.1
```

## Finding Your Device Paths

Before installing, identify the correct device paths on your node:

```bash
# List all USB serial devices
ls -la /dev/ttyUSB* /dev/ttyACM* 2>/dev/null

# Check dmesg for recently plugged-in devices
dmesg | grep -i 'tty\|usb'

# Common mappings:
#   Zigbee dongle (e.g. ConBee II):  /dev/ttyUSB0 or /dev/ttyACM0
#   Z-Wave dongle (e.g. Aeotec):    /dev/ttyACM0 or /dev/ttyUSB0
#   Note: paths can change on reboot — consider udev rules for stable names
```

## Accessing the Dashboard

After installation:

- **With Ingress enabled**: Navigate to `https://<ingress.host>` (e.g. `https://home.example.com`)
- **With hostNetwork**: Navigate to `http://<node-ip>:8123`
- **Port forward** (any setup): `kubectl port-forward -n home-assistant svc/home-assistant 8123:80`
- **Z-Wave JS UI**: `http://<node-ip>:30091` (when using NodePort) or `kubectl port-forward -n home-assistant svc/zwavejs2mqtt 8091:8091`

## Upgrading

```bash
# Update repo
helm repo update

# Upgrade the release
helm upgrade my-ha home-assistant/home-assistant -f my-values.yaml

# Check status
helm status my-ha
kubectl get pods -n home-assistant
```

## Uninstalling

```bash
helm uninstall my-ha
```

PersistentVolumeClaims are **not** deleted automatically to prevent data loss. To remove them:

```bash
kubectl delete pvc -n home-assistant home-assistant-pvc zwavejs2mqtt-pvc
```

## Testing (CI)

A test values overlay is provided for CI environments (like KIND clusters) where no hardware is available:

```bash
helm install ha-test ./charts/home-assistant -f charts/home-assistant/values-test.yaml
```

This deploys into the `ha-test` namespace with sidecars and ingress disabled.

## Troubleshooting

### Pod stuck in CrashLoopBackOff

Check logs: `kubectl logs -n home-assistant deployment/home-assistant -c home-assistant`

Common causes:
- `configuration.yaml` syntax error — validate your `homeAssistant.configuration` value
- PVC not binding — check `kubectl get pvc -n home-assistant` and ensure a StorageClass is available

### Bluetooth not working

Ensure:
- `homeAssistant.bluez.enabled: true`
- `homeAssistant.devices.dbus.enabled: true`
- The node has a Bluetooth adapter (`hciconfig` on the node)
- The pod runs in privileged mode (`homeAssistant.privileged: true`)

### Z-Wave device not found

Ensure:
- The Z-Wave USB dongle is plugged into the node where the pod is scheduled
- `zwavejs.devices.zwave.path` matches the actual device path on the host
- Consider using `nodeSelector` or `nodeAffinity` to pin the pod to the correct node
