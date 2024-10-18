
# **Understanding Docker, Containerd, and CLI Tools in Kubernetes**

> This document explains the evolution of container runtimes, focusing on the differences between Docker and Containerd. We also explore various CLI tools—`ctr`, `nerdctl`, and `crictl`—used to interact with these runtimes in Kubernetes.

---

## **1. Evolution of Container Runtimes**

### **Docker: The Beginning of the Container Era**
- **Docker** revolutionized containerization by simplifying the process of building, deploying, and managing containers.
- **Initial setup**: Kubernetes was originally designed to orchestrate Docker containers, making Docker the first supported runtime.
- **Tight Coupling**: Early Kubernetes versions only supported Docker, leading to a close integration between the two technologies.

### **The Rise of Other Runtimes & Introduction of CRI**
- **Container Runtime Interface (CRI)**: As Kubernetes grew, other container runtimes (like rkt) wanted to integrate with it.
- **CRI’s Role**: CRI allowed multiple runtimes to work with Kubernetes as long as they adhered to the **Open Container Initiative (OCI)** standards:
  - **ImageSpec**: Defines how an image should be built.
  - **RuntimeSpec**: Sets standards on how container runtimes should behave.

---

## **2. Docker’s Unique Position and the Dockershim**

### **Docker’s Initial Limitation**
- **Pre-CRI Docker**: Docker was not built to follow the CRI standards.
- **Dockershim**: Kubernetes introduced **dockershim** to allow continued support for Docker. It acted as a temporary bridge between Docker and Kubernetes.
  
### **Docker’s Complete Ecosystem**
Docker was more than just a runtime. It included:
  - **Docker CLI**: Interface for managing containers.
  - **Docker API**: Exposed endpoints for interacting with Docker.
  - **Build Tools**: Tools to build container images.
  - **runC**: The core container runtime.
  - **Containerd**: A higher-level container manager built on top of runC.

---

## **3. The Emergence of Containerd**

### **Containerd’s Role**
- **Separation from Docker**: Although part of Docker, **Containerd** evolved as an independent project under the **Cloud Native Computing Foundation (CNCF)**.
- **CRI Compatibility**: Unlike Docker, Containerd is fully CRI-compatible, making it easier to work with Kubernetes.
  
### **Removing Dockershim (Kubernetes v1.24)**
- **End of Dockershim**: Kubernetes v1.24 removed the dockershim, officially ending support for Docker as a runtime.
- **OCI Compliance**: Docker images still work because Docker followed the OCI ImageSpec standards, making them compatible with Containerd.

---

## **4. CLI Tools for Containerd & Kubernetes**

### **4.1. ctr (Containerd CLI)**
- **Purpose**: A basic tool that comes with Containerd, primarily for **debugging**.
- **Limitations**: It has a limited set of features and is **not user-friendly** for general container management.
- **Usage Example**:
  ```bash
  # Pull an image
  ctr images pull redis:latest
  
  # Run a container
  ctr run redis:latest redis-container
  ```
  
### **4.2. nerdctl (Containerd’s User-Friendly CLI)**
- **Docker-like CLI**: **nerdctl** is designed to be a **Docker-like** CLI for Containerd, providing a much better user experience.
- **Key Features**:
  - **Encrypted container images**.
  - **Lazy pulling of images**.
  - **P2P image distribution**.
  - **Namespaces** in Kubernetes.
  
- **Usage Example**:
  ```bash
  # Running a container with nerdctl
  nerdctl run -d -p 8080:80 nginx:latest
  
  # Working with Kubernetes Namespaces
  nerdctl --namespace k8s.io run -d nginx:latest
  ```
  
### **4.3. crictl (Kubernetes CRI CLI)**
- **Designed by Kubernetes Community**: This tool interacts with **CRI-compatible** runtimes, like Containerd, from a **Kubernetes perspective**.
- **Primarily for Debugging**: While you can manage containers with it, its primary use is for **debugging purposes**.
  
- **Usage Example**:
  ```bash
  # List pods
  crictl pods
  
  # Get container logs
  crictl logs <container-id>
  
  # Running commands inside a container
  crictl exec -i -t <container-id> /bin/bash
  ```

---

## **5. CLI Tool Comparison**

| **Command** | **Docker**        | **nerdctl (Containerd)** | **crictl (CRI)** |
|-------------|-------------------|--------------------------|------------------|
| `ps`        | List containers   | Yes                      | Yes              |
| `run`       | Create container  | Yes                      | Yes              |
| `logs`      | View logs         | Yes                      | Yes              |
| `exec`      | Run command in container | Yes              | Yes              |
| `pull`      | Pull images       | Yes                      | Yes              |

> **Note**: `ctr` is not included because it is primarily used for debugging purposes and lacks many user-friendly features.

---

## **6. Changes in Kubernetes v1.24**
- **Deprecated Dockershim**: The **dockershim** endpoint was replaced by **cri-dockerd.sock**, and Kubernetes officially stopped supporting Docker as a runtime.
- **Impact**: Users must now configure the CRI endpoints manually, as documented in [Kubernetes GitHub Pull Request #869](https://github.com/kubernetes/kubernetes/pull/869).

---

## **7. Summary**
- **ctr**: Debugging tool for Containerd.
- **nerdctl**: A user-friendly, Docker-like CLI for Containerd, recommended for general-purpose container management.
- **crictl**: A CLI from the Kubernetes community for debugging and interacting with any CRI-compatible runtimes.
  
Containerd has become the go-to runtime for Kubernetes deployments, and tools like `nerdctl` and `crictl` offer more flexibility and efficiency for managing and debugging containers within modern cloud environments.

---

> **Pro Tip**: If you're transitioning from Docker to Kubernetes, learning `nerdctl` is a great way to continue using familiar Docker commands while taking advantage of the newer Containerd features.
