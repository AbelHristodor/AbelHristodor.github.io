---
layout: post
title: Securing Docker Containers
subtitle: A guide to secure docker containers
cover-img: /assets/img/docker/containers.jpg
thumbnail-img: /assets/img/docker/secure-logo.png
share-img: /assets/img/docker/secure-logo.png
comments: true
tags: [containers, infrastructure, docker]
author: Abel Hristodor
---

## Writing Secure Docker Containers: Best Practices and Strategies

Containerization has become a **cornerstone** of modern software development, offering a *lightweight*, *scalable*, and *efficient* solution for deploying applications.

[Docker](https://www.docker.com/) is one of the most popular container platforms, but with its convenience comes the need for careful security considerations. Ensuring your Docker containers are **secure** is essential to protect against *vulnerabilities*, *mitigate risks*, and *ensure the integrity* of your applications.

Below are key practices and strategies for writing secure Docker containers, ranging from user permissions to container image scanning.

---

#### 1. **Run Containers as Non-Root Users**

By default, Docker containers run with **root privileges**, which poses a significant **security risk** if a container is compromised. If an attacker gains access to a container running as root, they may also gain root access to the **host system**, leading to serious security breaches.

Configuring containers properly to run as a non-root user reduces this risk.
You can do this by adding a user to your Dockerfile:

```dockerfile
# Dockerfile
FROM ubuntu:latest
RUN useradd -m -s /bin/bash appuser
USER appuser
```

This example creates a user `appuser` and sets it as the default user for all subsequent commands. Always avoid running applications with root privileges unless absolutely necessary.

---

#### 2. **Leverage Multi-Stage Builds for Smaller, More Secure Images**

Multi-stage builds in Docker help **reduce the size** of your final container image by separating the **build** environment from the **runtime** environment.

Here’s an example of a multi-stage build:

```dockerfile
# Dockerfile
# Build stage
FROM golang:alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Production stage
FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/myapp /app/
CMD ["./myapp"]
```

In this example, the Go application is built in the first stage (`builder`), and the final image contains only the compiled binary and necessary runtime dependencies. By **excluding development dependencies**, you reduce the image size and potential vulnerabilities.

---

#### 3. **Implement Health Checks in the Dockerfile**

To ensure your application is running correctly, Docker provides a [HEALTHCHECK](https://docs.docker.com/reference/dockerfile/#healthcheck) directive in the [Dockerfile](https://docs.docker.com/reference/dockerfile/). Health checks periodically test the **status** of a running container and can help identify when an application is *unresponsive* or *failing*.

Here’s how you can add a health check to your `Dockerfile`:

```dockerfile
# Dockerfile
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
  CMD curl --fail http://localhost:8080/health || exit 1
```

In this example, Docker checks the `/health` endpoint of the application every 30 seconds. If the application does not respond successfully after three retries, the container is considered unhealthy, allowing orchestrators like Kubernetes to restart the container.

---

#### 4. **Use Distroless Images**

Distroless images are **minimal base images** that only include the **essential** libraries required to run your application.

They exclude unnecessary tools like package managers, shells, or debuggers, which significantly reduces the attack surface and helps protect against vulnerabilities.

Google offers a set of [Distroless images](https://github.com/GoogleContainerTools/distroless) that are optimized for security. Here’s an example of using a distroless base image:

```dockerfile
# Dockerfile
FROM golang:1.20 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Distroless production stage
FROM gcr.io/distroless/base:nonroot
COPY --from=builder /app/myapp /app/
CMD ["/app/myapp"]
```

---

#### 5. **Set Volumes and File System Permissions to Read-Only**

To minimize the risks associated with file system manipulation in a container, it's important to configure **volumes** and **directories** with the *least privilege* necessary. Whenever possible, mount directories as **read-only** to prevent unauthorized writes or modifications to sensitive data.

Here’s how you can define a volume with read-only permissions:

```dockerfile
# Dockerfile
VOLUME /data
COPY myapp /app/
CMD ["./myapp"]
```

Then, when running the container:

```bash
docker run -v /data:/data:ro mycontainer
```

This mounts the `/data` directory as read-only (`ro`), restricting any write operations within the container and preventing accidental or malicious data alteration.

Additionally, setting proper file permissions in the container is essential. Ensure sensitive files have restrictive access settings:

```dockerfile
# Set file permissions to read-only for sensitive files
RUN chmod 400 /app/config.json
```

---

#### 6. **Regularly Scan and Verify Container Images**

Container images can contain *vulnerabilities*, especially when built from **outdated** base images or dependencies. To address this, it’s essential to regularly **scan your container images** for vulnerabilities and apply updates to patched versions of libraries or dependencies.

Several tools are available for scanning Docker images, including:

- **Clair**: An open-source project that analyzes Docker images for vulnerabilities.
- **Trivy**: A comprehensive vulnerability scanner for container images, Git repositories, and Kubernetes clusters.

Here’s an example of scanning an image with Trivy:

```bash
trivy image mycontainer:latest
```

Running these scans as part of your CI/CD pipeline ensures that images are up to date and secure before deployment.

---

![DockerBadge](https://iamnorte.com/assets/img/docker/secure-logo.png){: .mx-auto.d-block :}

### Conclusion

Writing secure Docker containers requires a combination of strategies to reduce the attack surface and mitigate risks.

By running containers as *non-root* users, using *multi-stage* builds, incorporating *health checks*, adopting *distroless* images, enforcing strict file system permissions, and regularly **scanning** images, you can greatly enhance the security of your dockerized applications.

Security is an **ongoing** process, and adopting these practices will help **maintain** a strong **security posture** in your containerized environment.
