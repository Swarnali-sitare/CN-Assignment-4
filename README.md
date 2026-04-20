# Distance-vector router

This directory contains only the assignment sources used to build and run the router daemon.

## Contents

| File | Role |
|------|------|
| `router.py` | UDP/JSON distance-vector daemon; updates Linux routes with `ip route` |
| `Dockerfile` | Alpine image with Python 3 and `iproute2` |

---

## Prerequisites

- Docker
- Linux host (or Linux VM) if you rely on `ip route` inside the container — the daemon needs `NET_ADMIN` to install routes

---

## 1. Build the image

```bash
docker build -t dv-router .
```

---

## 2. Run one container

The process must be allowed to change routes:

```bash
docker run --rm -it \
  --cap-add=NET_ADMIN \
  --sysctl net.ipv4.conf.all.rp_filter=0 \
  --sysctl net.ipv4.conf.default.rp_filter=0 \
  -e MY_IP=10.0.1.2 \
  -e NEIGHBORS=10.0.1.3 \
  dv-router
```

Use `sudo docker` if your user cannot talk to the Docker daemon.

A **single** instance with no real neighbors will not show full routing behavior; you need at least one peer running the same image with matching addresses.

---

## 3. Assignment triangle (three routers)

The full assignment topology needs **three** routers on **three** `/24` subnets with interfaces and `NEIGHBORS` wired to match that graph. Build the same `dv-router` image from this folder, then start three containers with:

- distinct `MY_IP` values on the correct subnets,
- `NEIGHBORS` listing the peer IPs each router talks to on UDP port `5000`,
- `DV_FORCE_LOCAL_SUBNETS` listing every `/24` that is directly connected on that host (multi-homed routers need both subnets).

After convergence, test with `ping` and `ip route` inside the containers. If ICMP between subnets on Docker bridges fails unexpectedly on your host, search for **`bridge-nf-call-iptables`** — disabling it on the host is a common lab fix.
