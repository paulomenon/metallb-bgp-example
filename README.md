[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

# MetalLB BGP Example

This repository provides example configurations to deploy and run **MetalLB in BGP mode** on an OpenShift or Kubernetes cluster. It demonstrates how to create IPAddressPools, configure BGP peers, and advertise external IPs using `BGPAdvertisement`.

---

## 📌 What This Repository Covers

- Configuration of MetalLB using the native Kubernetes CRDs.
- Setup of **IPAddressPools** for load-balanced services.
- Definition of **BGPPeers** to integrate with your router or network infrastructure.
- Activation of IP advertisement via **BGPAdvertisement**.
- Basic usage and testing with a LoadBalancer service.

---

## 🛠 Prerequisites

- OpenShift 4.12+ or upstream Kubernetes 1.20+
- MetalLB installed and configured with `bgp` controller
- BGP-capable router on your network or a virtual lab (e.g., FRR, MikroTik, Cilium lab)
- Cluster network configured to allow BGP peering

---

## 🧾 Repository Structure

```
metallb-bgp-example/
├── bgp-peer.yaml           # Defines the external BGP peer (router)
├── bgp-advertisement.yaml  # Enables advertisement for an IP pool
├── ipaddresspool.yaml      # Pool of IPs assigned to LoadBalancer services
└── sample-service.yaml     # Test service to validate external IP assignment
```

---

## 🚀 How to Use

1. Clone the repo:

   ```bash
   git clone https://github.com/paulomenon/metallb-bgp-example.git
   cd metallb-bgp-example
   ```

2. Apply the MetalLB configurations:

   ```bash
   oc apply -f ipaddresspool.yaml
   oc apply -f bgp-peer.yaml
   oc apply -f bgp-advertisement.yaml
   ```

3. Deploy a sample LoadBalancer service:

   ```bash
   oc apply -f sample-service.yaml
   ```

4. Watch the external IP get advertised:

   ```bash
   oc get svc nginx-lb
   ```

---

## 🔄 BGP Traffic Flow

```
User Request
     ↓
BGP Router ←→ MetalLB Speaker (BGP Peer)
     ↓
Cluster Node Network
     ↓
LoadBalancer Service IP (from MetalLB pool)
     ↓
Application Pod
```

> MetalLB uses BGP to tell the router: “I have this IP address, route it to me.”

---

## 🔐 Sample Configurations

### `ipaddresspool.yaml`

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: bgp-pool
  namespace: metallb-system
spec:
  addresses:
    - 192.168.100.240-192.168.100.250
```

### `bgp-peer.yaml`

```yaml
apiVersion: metallb.io/v1beta1
kind: BGPPeer
metadata:
  name: my-router
  namespace: metallb-system
spec:
  myASN: 64500
  peerASN: 64501
  peerAddress: 192.168.100.1
```

### `bgp-advertisement.yaml`

```yaml
apiVersion: metallb.io/v1beta1
kind: BGPAdvertisement
metadata:
  name: bgp-ad
  namespace: metallb-system
spec:
  ipAddressPools:
    - bgp-pool
```

---

## 🧪 Test It

- Create a DNS entry pointing to an IP from the pool
- Curl the IP or DNS name externally to verify routing works
- Check your router’s BGP table to confirm MetalLB announced the route

---

Maintained by [@paulomenon](https://github.com/paulomenon). Contributions welcome!
