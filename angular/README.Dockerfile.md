# Dockerfile Explanation
- [Summary](#summary)
- [Details](#details)
  - [Multi-stage build](#multi-stage-build)
  - [Multi-platform build](#multi-platform-build)
    - [BUILDPLATFORM](#buildplatform)
  - [Working directory](#working-directory)
  - [Alpine images](#alpine-images)
  - [Nginx](#nginx)
    - [Benefits of using Nginx in this context](#benefits-of-using-nginx-in-this-context)
    - [About the CMD](#about-the-cmd)
  - [Running as a non-root user](#running-as-a-non-root-user)

# Summary
The `Dockerfile` used in this project aim to apply best practices, ensuring that the application not only runs seamlessly in different environments, but also that the build time is fast and the final image size is optimized. This README will provide a detailed explanation of those aspects.

# Details
## Multi-stage build
A multi-stage build in Docker allows you to use multiple `FROM` statements in a single Dockerfile, enabling separation between the build environment and the final runtime image. This approach has several advantages:

- Smaller final images: By copying only the necessary build output into the final stage, you avoid including SDKs, source code, or temporary files. The final image is more efficient since it includes only what's required to run the application.

- Faster builds: Enable layer caching, steps like `npm install` can be cached as long as the project file hasn’t changed, reducing rebuild time significantly.

## Multi-platform build
### BUILDPLATFORM
`BUILDPLATFORM` is a special Docker `automatic platform argument` that represents the platform where the build is taking place, not where the final image will run.
It's useful when building images in a cross-compilation scenario—when the machine doing the build differs from the target platform.

By default, `BUILDPLATFORM` is set to the platform of the machine executing the build.
For example:

- `linux/amd64` – Most desktop and laptop computers, cloud servers like AWS EC2, Azure VMs, Google Cloud.

- `linux/arm64` – Apple M1/M2 Macs, Raspberry Pi 4 (64-bit OS), ARM-based cloud servers (e.g., AWS Graviton).

- `linux/arm/v7`– Raspberry Pi 2/3 running 32-bit OS.

- `windows/amd64` – Windows Server containers.

To simulate building on a different platform, you can override the default using the `--platform` flag in the docker build command:

```bash
docker buildx build --platform=<target_platform> -t <app_name>:<tag> .
```

For example, if you're using a MacBook Pro M4 with `linux/arm64`, and you want to build an image for an Ubuntu machine on AWS with `linux/amd64`, you can run the following command:

```bash
docker buildx build --platform=linux/amd64 -t angular-server:latest .
```

## Working directory
The `WORKDIR` instruction in a Dockerfile sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY`, and `ADD` instructions that follow it. Although not required to build the image, it is a common Dockerfile practice that ensures a consistent and predictable file path context throughout the build and runtime stages.

For example, with `WORKDIR /app`, the final source code of the application will be placed in the /app folder, which helps us quickly identify the source code directory inside the container. If we do not use the `WORKDIR` instruction, the image will still build successfully and the container will still run. However, when navigating the file structure from the root directory of the container, it will be cluttered with system folders (such as /bin, /dev, /etc, etc.), making it harder to locate the application files.

## Alpine images
Alpine Linux is a security-oriented, lightweight Linux distribution designed for resource efficiency. It is widely used in Docker images to minimize image size and reduce potential security vulnerabilities. In this Dockerfile, both the Node.js build stage and the Nginx production stage use Alpine-based images.

Using Alpine has several advantages:

- **Smaller image size**: Alpine images are significantly smaller (typically under 10 MB) compared to their Debian or Ubuntu-based counterparts. This results in faster image downloads, reduced bandwidth consumption, and quicker container startup times.

- **Security**: The smaller footprint means fewer installed packages, which inherently reduces the attack surface. Additionally, Alpine receives regular security updates.

- **Performance and efficiency**: Due to its lightweight nature, Alpine requires fewer resources at runtime, which is particularly valuable in resource-constrained environments like CI/CD pipelines or edge computing.

However, it's important to be aware of potential caveats when using Alpine:

- **Compatibility issues**: Some Node.js dependencies that rely on native binaries may fail to compile or require additional build tools when running on Alpine, since it uses `musl` instead of `glibc`.

- **Extra dependencies**: To install some native modules or build Angular applications, you may occasionally need to install additional packages like `python3`, `make`, or `g++`.

To address this, you can modify the Dockerfile to install required Alpine packages during the build stage:

```Dockerfile
RUN apk add --no-cache python3 make g++
```

## Nginx
Nginx is a high-performance web server that is commonly used to serve static files, making it an ideal choice for hosting Angular applications in production. In this Dockerfile, the final production stage uses the [unprivileged Nginx](#running-as-a-non-root-user) image to serve the output of the Angular build.

By copying the compiled Angular files into Nginx’s default serving directory (`/usr/share/nginx/html`), we ensure the application is delivered efficiently and reliably via HTTP.

### Benefits of using Nginx in this context:

- **Efficient static file serving**: Nginx is optimized to serve static assets (HTML, CSS, JS, images) with low latency and high throughput.

- **Built-in production defaults**: The default Nginx configuration in the official image is already tuned to serve static files securely and efficiently.

### About the CMD

The command `CMD ["nginx", "-g", "daemon off;"]` is included at the end of the Dockerfile to keep the Nginx process running in the foreground. While it is **not strictly required**, because the base `nginx` image already defines this as its default `CMD`, including it explicitly in the Dockerfile has some advantages:

- **Clarity**: It makes it immediately obvious to the reader or maintainer how the container is intended to run.
- **Extensibility**: It allows for easier customization in the future if you need to pass additional Nginx flags or replace it with a different entrypoint.

The `-g "daemon off;"` option tells Nginx not to run in the background (as a daemon), which is necessary inside containers to keep the main process in the foreground and prevent the container from exiting.

## Running as a non-root user
`nginxinc/nginx-unprivileged:alpine-perl` is an **unprivileged** Nginx image, which means Nginx serves the application with `non-root` user permissions instead of `root`. This is an important security best practice, as running processes as the `root` user can pose significant security risks.

Note that because we use the `nginxinc/nginx-unprivileged:alpine-perl` image, the default NGINX listen port is now 8080 instead of 80. For more information about this image, see: [nginx-unprivileged](https://hub.docker.com/r/nginxinc/nginx-unprivileged)