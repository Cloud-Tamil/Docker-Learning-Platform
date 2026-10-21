# 🌐 Pinger App: End-to-End Docker Workflow

A lightweight, hands-on DevOps project demonstrating the foundational container lifecycle: writing custom Dockerfiles, understanding filesystem layers (`COPY` vs. `ADD`), image building, multi-tagging strategies, and registry distribution.

---

## 📌 Architectural Overview

```text
  [ Project Context ]          [ Build Stage ]                 [ Runtime / Distribution ]
  ├── Dockerfile         ───►  docker build -t pinger .  ───►  docker run my-ping
  ├── copy_file.txt                                      ───►  docker tag pinger:v1
  └── add_file.txt                                       ───►  docker push local-registry:5000/pinger:v1
```

---

## 🚀 Quickstart Walkthrough

### 1. Workspace Preparation & Build Context
Initialize your working directory and sample configuration files:

```bash
mkdir -p /root/pinger && cd /root/pinger

# Create payload artifacts
echo "Pinger application file copied with COPY" > copy_file.txt
echo "Pinger configuration file added with ADD" > add_file.txt
```

### 2. Dockerfile Specification
Create the minimal, deterministic container specification:

```dockerfile
# /root/pinger/Dockerfile
FROM debian:bookworm-slim

# Install dependencies with cache cleanup to minimize footprint
RUN apt-get update \
    && apt-get install -y --no-install-recommends bash iputils-ping \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Stage build artifacts into the container image
COPY copy_file.txt /app/
ADD add_file.txt /app/

# Default ping routine
CMD ["ping", "-c", "4", "killercoda.com"]
```

### 3. Build & Local Execution
Compile the image context and spin up a test container:

```bash
# Build base image
docker build -t pinger .

# Execute test ping container (terminates automatically after 4 packets)
docker run --name my-ping pinger

# Inspect container state and terminal output
docker ps -a
docker logs my-ping
```

### 4. Build Artifact Inspection
Verify that filesystem operations placed files correctly inside the image:

```bash
# Inspect contents and verify COPY/ADD layers
docker run --rm --entrypoint ls pinger -l /app
docker run --rm --entrypoint cat pinger /app/copy_file.txt
docker run --rm --entrypoint cat pinger /app/add_file.txt
```

### 5. Tagging & Registry Distribution
Publish versioned artifacts to your local registry:

```bash
# 1. Tag image for versioning and registry destination
docker tag pinger pinger:v1
docker tag pinger local-registry:5000/pinger:v1

# 2. Verify registry connectivity
curl -sS http://local-registry:5000/v2/

# 3. Push artifact to the registry
docker push local-registry:5000/pinger:v1

# 4. Pull and run image directly from registry to verify integrity
docker run --rm local-registry:5000/pinger:v1
```

---

## 🛠️ Key Concepts & Technical Notes

| Concept | Purpose & Distinction |
| :--- | :--- |
| **`COPY` vs. `ADD`** | `COPY` is preferred for straightforward local file insertion. `ADD` includes additional features like auto-extracting `.tar` archives and downloading from remote URLs. |
| **Slim Base Images** | `debian:bookworm-slim` provides a minimal attack surface while allowing explicit package installations (`iputils-ping`). |
| **Transient Containers** | Using `--rm` removes intermediate verification containers immediately upon process completion, keeping the host clean. |

---

## 📋 Docker Command Cheat Sheet

```bash
# Build & Lifecycle
docker build -t <name>:<tag> .          # Build image from local Dockerfile
docker run --name <name> <image>        # Instantiate and run container
docker run --rm <image>                 # Run transient container and purge on exit

# Diagnostics & Inspection
docker ps -a                            # List all container states (active & exited)
docker image ls                         # View local image inventory and digest IDs
docker logs <container-name>            # Stream standard output logs

# Registry Operations
docker tag <source> <target>            # Apply alias/registry tag
docker push <registry>/<image>:<tag>    # Upload image to target registry
docker pull <registry>/<image>:<tag>    # Retrieve image from target registry
```

---

## 🧹 Maintenance & Cleanup

To wipe the sandbox environment clean:

```bash
docker rm -f my-ping 2>/dev/null || true
docker rmi pinger pinger:v1 local-registry:5000/pinger:v1 2>/dev/null || true
```
