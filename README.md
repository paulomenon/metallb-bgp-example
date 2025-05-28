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

### `f5_ip_address_pool.yaml`

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: f5-ip-pool
  namespace: metallb-system
spec:
  addresses:
    - 192.168.2.10-192.168.2.50 	# A pool of IPs you want MetalLB to assign
  autoAssign: true                # MetalLB will automatically assign IPs from this range
  avoidBuggyIPs: false            # BGP for routing IPs to your network
```

### `f5_bgp_peer.yaml`

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
### `bgp_peer_failover.yaml`

```yaml
apiVersion: metallb.io/v1beta2
kind: BGPPeer
metadata:
  name: f5-bgp-peer
  namespace: metallb-system
spec:
  peerAddress: 192.168.2.1     # IP of the F5 (or upstream BGP router)
  peerASN: 65001           	# ASN of the F5 BGP speaker
  myASN: 64512             	# ASN assigned to MetalLB
  bfdprofile: default  # Optional: BFD profile for fast failure detection
```

### `f5_bgp_advertisement.yaml`

```yaml
kind: BGPAdvertisement
apiVersion: metallb.io/v1beta1
metadata:
  name: advertise-bgp-pool
  namespace: metallb-system
spec:
  ipAddressPools:
  - f5-ip-pool
```

### `f5_L2_advertisement.yaml`
This is optional and not used in this configuration, as the setup is designed for Layer 3 external load balancing.

---

## 🧪 Test It

- Create a DNS entry pointing to an IP from the pool
- Curl the IP or DNS name externally to verify routing works
- Check your router’s BGP table to confirm MetalLB announced the route

---

Maintained by [@paulomenon](https://github.com/paulomenon). Contributions welcome!
