
# Enablement Recommendations: Container 101

## Goals and Why It Matters

The goal is to understand the core concepts of containers, as Ansible Automation Platform (AAP) uses them for its entire architecture. This section connects the tools to their specific purpose within the AAP ecosystem.

* **Run Containers (Podman):** Learn to use **Podman** to run and manage containers. This is your primary tool for *local development*. It allows you to run and test an **Execution Environment (EE)** on your laptop, ensuring your automation works *before* pushing it to AAP.
* **Build Containers (Buildah):** Understand how to use **Buildah** to build new container images from a Containerfile. This is a critical skill for *customizing* EEs, such as adding new Python libraries (like `boto3`) or Ansible Collections that your automation requires.
* **Store & Distribute Containers (Quay):** Grasp the concept of a **Container Registry** (like Quay.io). This is the "source of truth" for container images. You will push your custom EEs (built with Buildah) to a registry so that AAP can pull them down to run your automation jobs.

## RHLS Core Recommendations

* [Red Hat OpenShift Development I: Introduction to Containers with Podman (DO188)](https://www.redhat.com/en/services/training/do188-red-hat-open-shift-development-introduction-containers-with-podman)

## Independent Learning Path Recommendations

### Labs

* [Overview: Container Fundamentals](https://developers.redhat.com/learn/rhel/container-fundamentals)  
* [Write your first Containerfile for Podman](https://www.redhat.com/en/blog/write-your-first-containerfile-podman)

### Reading Material and References

* [Understanding Containers](https://www.redhat.com/en/topics/containers)  
* [What is Buildah?](https://www.redhat.com/en/topics/containers/what-is-buildah)
* [What is Podman?](https://www.redhat.com/en/topics/containers/what-is-podman)
* [3 advantages of Podman vs. Docker](https://developers.redhat.com/articles/2023/08/03/3-advantages-docker-podman)
* [What is a Container Registry?](https://www.redhat.com/en/topics/cloud-native-apps/what-is-a-container-registry)
* [What is a Linux container?](https://www.redhat.com/en/topics/containers/whats-a-linux-container)

### eBooks

* [Podman in Action](https://developers.redhat.com/e-books/podman-action)

---

## Sanity Check

### Key Commands to Know

Here are the essential commands for Podman and Buildah that align with your goals.

| Tool | Command | What it does |
| :--- | :--- | :--- |
| **Podman** | `podman pull [image]` | Downloads an image from a registry. |
| | `podman images` | Lists all container images on your local machine. |
| | `podman run [image]` | Runs a command in a new container. |
| | `podman ps` | Lists *running* containers. |
| | `podman ps -a` | Lists *all* containers, including stopped ones. |
| | `podman rm [container]` | Removes a stopped container. |
| | `podman rmi [image]` | Removes a local image. |
| **Buildah** | `buildah bud -f [file] -t [tag] .` | **Builds** a new container image from a `Containerfile`. |
| | `buildah login [registry]` | Logs you into a remote container registry. |
| | `buildah push [image] [registry]` | **Pushes** your newly built image to a remote registry (like Quay). |

### What is an Execution Environment (EE)?

You've learned that AAP uses containers, but specifically, it uses a concept called **Execution Environments (EEs)**.

An **Execution Environment** is a container image (built with tools like **Buildah** and run by **Podman**) that bundles everything needed to run your automation. This includes:

* A specific version of `ansible-core`.
* Any required Python libraries (e.g., `boto3` for AWS).
* Any required Ansible Collections (e.g., `community.aws`).
* The RHEL Universal Base Image (UBI) as its foundation.

**Why this matters:** Instead of managing Python virtual environments on the server, you now manage a single, version-controlled **EE image**. When you build a custom EE, you push it to a **registry** (like **Quay**) so AAP can pull and use it to run your jobs.

### Your First "Hello World" (Podman)

This exercise will confirm you have Podman installed and can run a container. You will pull the RHEL Universal Base Image (UBI) and run a simple command inside it.

1. **Pull the UBI image:**

    ```bash
    podman pull [registry.access.redhat.com/ubi9/ubi-minimal](https://registry.access.redhat.com/ubi9/ubi-minimal)
    ```

2. **Run the container:** This command starts a new container from the `ubi-minimal` image, runs the `cat /etc/os-release` command inside it, and then removes the container.

    ```bash
    podman run --rm ubi9/ubi-minimal cat /etc/os-release
    ```

3. **Expected Output:** You should see the contents of the RHEL release file, proving you ran a command *inside* a container.

    ```bash
    NAME="Red Hat Enterprise Linux"
    VERSION="9.x (Plow)"
    ID="rhel"
    ID_LIKE="fedora"
    VERSION_ID="9.x"
    PLATFORM_ID="platform:el9"
    ...
    ```

4. **List local images:** See the image you downloaded.

    ```bash
    podman images
    ```
