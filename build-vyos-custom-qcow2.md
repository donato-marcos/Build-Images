
# Building a VyOS ISO and Base qcow2 Image

This document describes how to build a **custom VyOS ISO** and, from it, create a base `.qcow2` image for use with Libvirt/KVM/QEMU.


## 🧱 Prerequisites

- **Docker** installed and working
- Internet access to download repositories and dependencies
- Approximately **10–15 GB** of disk space
- Linux (any distribution with Docker support)

---

## 🐳 Install Docker

```bash
curl -fsSL https://get.docker.com -o get-docker.sh

# Dry-run mode (optional, to preview actions)
sudo sh get-docker.sh --dry-run

# Actual installation
sudo sh get-docker.sh

# Add your user to the docker group
sudo groupadd docker 2>/dev/null || true
sudo usermod -aG docker $USER

# Apply group changes (or restart your shell)
newgrp docker

# Enable and start services
sudo systemctl enable --now docker.service
sudo systemctl enable --now containerd.service

# Test installation
docker run hello-world
```



## 🔧 Build the VyOS ISO

```bash
# 1. Clone the build repository
git clone -b current --single-branch https://github.com/vyos/vyos-build
cd vyos-build

# 2. Pull the build Docker image
docker pull vyos/vyos-build:current

# 3. Run the build container
docker run --rm -it --privileged \
  -v $(pwd):/vyos \
  -w /vyos \
  vyos/vyos-build:current bash
```

> `You will be inside the container. All following commands are executed **inside the container**`.

```bash
# 4. Clean previous builds (if any)
sudo make clean

# 5. Build the ISO
sudo ./build-vyos-image \
  --architecture amd64 \
  --build-by "your-email@example.com" \
  --custom-package cloud-init \
  --custom-package qemu-guest-agent \
  generic
```

### 📌 Parameter explanation

| Parameter                           | Description                                     |
| ----------------------------------- | ----------------------------------------------- |
| `--architecture amd64`              | 64-bit architecture                             |
| `--build-by`                        | Build identifier (use your email)               |
| `--custom-package cloud-init`       | Adds cloud-init support (useful for automation) |
| `--custom-package qemu-guest-agent` | Adds integration with QEMU/KVM                  |
| `generic`                           | Default build type                              |

---

### 6. Locate the generated ISO

After a successful build, the ISO will be located at:

```bash
build/live-image-amd64.hybrid.iso
```

Outside the container, it will be available in:

```
vyos-build/build/
```

Copy the ISO to a safe location.

---

## 💿 Creating the base `.qcow2` image from the ISO

Now that you have the custom ISO, follow these steps to create an installed disk image:

1. Create a VM using the VyOS ISO (no networking required during installation)
2. Install VyOS normally onto the VM disk
3. After installation, shut down the VM
4. Locate the VM disk (usually a `.qcow2` file in your libvirt storage pool)
5. Make a copy of that file and rename it to `vyos-custom-image.qcow2`

This disk is already installed and ready to boot, making it suitable as a base image for Terraform with libvirt.

---

## 📁 Placing the image in the correct location

After creating `vyos-custom-image.qcow2`, move or copy it to your project's expected image directory:

```bash
cp vyos-custom-image.qcow2 /path/to/your/storage/pool/
```

---

## 🔍 Final verification

```bash
ls -lh /path/to/your/storage/pool/vyos-custom-image.qcow2
qemu-img info /path/to/your/storage/pool/vyos-custom-image.qcow2
```

---

## 📚 References

* [Docker Install](https://docs.docker.com/engine/install/)
* [VyOS Build documentation](https://docs.vyos.io/en/latest/contributing/build-vyos.html#build-iso)
* [VyOS Docker build image](https://hub.docker.com/r/vyos/vyos-build)
