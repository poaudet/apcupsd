# 🖥️ apcupsd Docker Stack

Monitor and manage your APC UPS using `apcupsd` in Docker. This stack supports both **master** (USB-connected) and **slave** (network-connected) configurations, enabling graceful shutdowns across multiple systems.

## 📦 Features

* **Master Mode**: Monitor a UPS connected via USB and broadcast status over the network.
* **Slave Mode**: Receive UPS status from a master and initiate graceful shutdowns.
* **Dockerized Deployment**: Easy setup using Docker and Docker Compose.
* **Portainer Compatible**: Manage your stack through Portainer's web interface.
* **Netdata Integration**: Monitor UPS metrics via Netdata or Netdata Cloud.

## 🚀 Getting Started

### Prerequisites

* Docker and Docker Compose installed on your system.
* An APC UPS connected via USB (for master setup).
* Network connectivity between master and slave systems.

### Clone the Repository

```bash
git clone https://github.com/poaudet/apcupsd.git
cd apcupsd
````

## 🛠️ Configuration

### Master Configuration (USB-Connected System)

1. **Docker Compose File**: Example `docker-compose.yml`:

```yaml
version: '3.7'

services:
  apcupsd:
    image: gregewing/apcupsd:latest
    container_name: apcupsd-master
    devices:
      - /dev/usb/hidraw0 # ← Update this for your device
    ports:
      - 3551:3551
    environment:
      - UPSNAME=ups.local
      - UPSCABLE=usb
      - UPSTYPE=usb
      - DEVICE=/dev/hidraw0 # ← Update this for your device
      - NETSERVER=ON
      - NISIP=0.0.0.0
      - TZ=America/Toronto # ← Update for your timezone
    # volumes:  # Uncomment to persist configuration
    #   - /etc/apcupsd:/etc/apcupsd
    restart: unless-stopped
```

2. **Start the Container**:

```bash
docker-compose up -d
```

---

### Slave Configuration (Network-Connected System)

1. **Docker Compose File**:

```yaml
version: '3.7'

services:
  apcupsd:
    image: gregewing/apcupsd:latest
    container_name: apcupsd-slave
    environment:
      - UPSNAME=from_master
      - UPSCABLE=ether
      - UPSTYPE=net
      - DEVICE=192.168.1.xxx:3551 # ← Replace with master's IP
      - TZ=America/Toronto # ← Update for your timezone
    # volumes:  # Uncomment to persist configuration
    #   - /etc/apcupsd:/etc/apcupsd
    restart: unless-stopped
```

2. **Start the Container**:

```bash
docker-compose up -d
```

---

## ✅ Verification

* **Check UPS Status**:

```bash
docker exec apcupsd-master apcaccess status
docker exec apcupsd-slave apcaccess status
```

* **Test Network Access**:

```bash
nc -vz 192.168.1.xxx 3551
```

Replace `192.168.1.xxx` with the master's IP address.

---

## 📊 Netdata Integration

To display UPS stats in Netdata:

1. **Ensure `go.d.plugin` is enabled**.

2. **Create the config**:

```bash
sudo tee /etc/netdata/go.d/apcupsd.conf > /dev/null <<EOF
jobs:
  - name: ups
    url: tcp://localhost:3551
EOF
```

3. **Restart Netdata**:

```bash
sudo systemctl restart netdata
```

*If you're using Netdata Cloud, your UPS stats will be visible there after local config updates.*

---

## 🔄 Graceful Shutdown

During power outage:

* **Master** detects it and broadcasts UPS status.
* **Slave** listens and shuts down cleanly when thresholds are met.
* No extra scripts required — `apcupsd` handles it all.

---

## 🧪 Testing the Setup

1. Unplug the UPS power cord.
2. Check `apcaccess status` on both containers.
3. Wait for the battery threshold; the slave should shut down automatically.

---

## 📝 Notes

* Keep your network gear on the UPS to maintain master-slave communication.
* Adjust thresholds and timing in `/etc/apcupsd/apcupsd.conf`.

---

## 📚 References

* [apcupsd Documentation](http://www.apcupsd.org/manual/manual.html)
* [gregewing/apcupsd Docker Image](https://hub.docker.com/r/gregewing/apcupsd)
