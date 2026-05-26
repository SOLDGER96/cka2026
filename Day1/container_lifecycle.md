# Container Lifecycle and Image Handling Architecture

![](/assets/cont_lifecycle.png)

The provided diagram illustrates the comprehensive lifecycle of containerized applications, dividing operations into two distinct but interconnected domains: **Image Handling** (managing the static, immutable templates) and **Container States** (managing the dynamic runtime instances). While applicable to Docker, the inclusion of the *Red Hat Container Catalog* specifically highlights enterprise environments typically managed via Podman.

---

## 1. Image Handling (Left Domain)

This section dictates how container images are acquired, stored, and shared between local infrastructure and external repositories.

### External vs. Local Storage
* **External Registries:** Images are hosted in remote repositories. These can be public (like Docker Hub or Quay.io), enterprise-grade (like the Red Hat Container Catalog), or internally hosted Private registries.
* **Local Storage (Node Local):** The local cache on the host server where images are stored after being downloaded, ready to be instantiated into containers.

### Image Management Commands
* **`pull`:** Fetches an image from an external registry down to the local node storage.
* **`push`:** Uploads a locally built or tagged image from the local storage up to an external registry.
* **`rmi` (Remove Image):** Deletes a specific image from the local storage, moving it to the "deleted" state to free up disk space.
* **`save` & `load`:** Allows images to be exported from local storage into an offline `.TAR` archive (`save`) and conversely imported from a `.TAR` archive back into local storage (`load`). This is critical for air-gapped environments.

---

## 2. Container States (Right Domain)

Once an image is available in local storage, it can be instantiated into a container. This section maps the various states a container can occupy and the commands used to transition between them.

### Instantiation
* **`run`:** The most common command. It takes an image from local storage, creates a container, and immediately starts it, transitioning it directly to the **running** state.
* **`create`:** Takes an image and prepares the container structure but leaves it in a **stopped** state, awaiting a future `start` command.

### State Transitions (Running, Paused, Stopped)
* **Running:** The active state where the containerized application is executing.
    * **`pause` / `unpause`:** Freezes the container's processes (using cgroups freezer) without destroying the container, transitioning it to the **paused** state and back.
    * **`stop` / `kill`:** Transitions a running container to the **stopped** state. `stop` sends a graceful SIGTERM, whereas `kill` sends an immediate SIGKILL.
* **Stopped:** The container is halted, but its filesystem and configuration remain intact on the host.
    * **`start` / `restart`:** Boots up a stopped container back into the **running** state.

### Resilience and Restart Policies
If a running container crashes (exited) or consumes too much memory and is terminated by the kernel (`OOM killed`), the system evaluates the **restart policy**. 
* **Yes:** If configured (e.g., `restart=always`), the daemon automatically attempts to transition it back to the **running** state.
* **No:** If no policy is set, the container falls back to the **stopped** state.

### Interactivity and Extraction
* **`exec`:** Allows administrators to spawn a secondary process (like a `/bin/bash` shell) inside an already **running** container for troubleshooting.
* **`commit`:** Captures the current state (filesystem changes) of a **running** container and saves it back to the local storage as a brand new image.
* **`export`:** Flattens the filesystem of a **running** container directly into a `.TAR` archive, stripping out image history.

### Destruction
* **`rm` (Remove):** Deletes a **stopped** container permanently.
* **`rm -f` (Force Remove):** Forcibly stops and deletes a **running** container in a single action, moving it to the deleted state.

---
### Summary
Mastering this lifecycle is foundational for infrastructure automation, troubleshooting, and drafting effective systemd unit files or playbooks. It provides a clear map of how an application moves from a remote repository, rests on local storage, executes as a container process, and is eventually cleaned up.