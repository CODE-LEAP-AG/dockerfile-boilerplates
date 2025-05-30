# Dockerfile Explanation
- [Summary](#summary)
- [Details](#details)
  - [Multi-stage build](#multi-stage-build)
  - [Multi-platform build](#multi-platform-build)
    - [BUILDPLATFORM](#buildplatform)
    - [TARGETARCH](#targetarch)
  - [Working directory](#working-directory)
  - [Running as a non-root user](#running-as-a-non-root-user)

# Summary
The `Dockerfile` used in this project aim to apply best practices, ensuring that the application not only runs seamlessly in different environments, but also that the build time is fast and the final image size is optimized. This README will provide a detailed explanation of those aspects.

# Details
## Multi-stage build
A multi-stage build in Docker allows you to use multiple `FROM` statements in a single Dockerfile, enabling separation between the build environment and the final runtime image. This approach has several advantages:

- Smaller final images: By copying only the necessary build output into the final stage, you avoid including SDKs, source code, or temporary files. The final image is more efficient since it includes only what's required to run the application.

- Faster builds: Enable layer caching, steps like `dotnet restore` can be cached as long as the project file hasn’t changed, reducing rebuild time significantly.

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

`docker buildx build --platform=<target_platform> -t <app_name>:<tag> .`

For example, if you're using a MacBook Pro M4 with `linux/arm64`, and you want to build an image for an Ubuntu machine on AWS with `linux/amd64`, you can run the following command:

`docker buildx build --platform=linux/amd64 -t dotnet-server:latest .`

### TARGETARCH
`TARGETARCH` is another Docker `automatic platform argument` that represents the architecture of the platform the image is being built for.

Common Values:
- `amd64` – for standard 64-bit Intel/AMD CPUs
- `arm64` – for 64-bit ARM architectures (like Apple Silicon or ARM-based servers)
- `arm` – for older 32-bit ARM platforms (e.g., Raspberry Pi)

#### Why You Should Not Manually Change TARGETARCH:
Docker automatically assigns `TARGETARCH` based on the platform you specify with --platform. For example, when you use `--platform=linux/amd64`, Docker automatically sets `TARGETARCH=amd64`, or when you use `--platform=linux/arm64`, Docker automatically sets `TARGETARCH=arm64`.

Technically, you can override this value manually using `--build-arg TARGETARCH=<your_target_arch>`. However, you should avoid doing this, as it can lead to inconsistencies between what your build tool generates and what Docker expects in the final image. This mismatch may result in runtime crashes or compatibility issues.

## Working directory
The `WORKDIR` instruction in a Dockerfile sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY`, and `ADD` instructions that follow it. Although not required to build the image, it is a common Dockerfile practice that ensures a consistent and predictable file path context throughout the build and runtime stages.

For example, with `WORKDIR /app`, the final source code of the application will be placed in the /app folder, which helps us quickly identify the source code directory inside the container. If we do not use the `WORKDIR` instruction, the image will still build successfully and the container will still run. However, when navigating the file structure from the root directory of the container, it will be cluttered with system folders (such as /bin, /dev, /etc, etc.), making it harder to locate the application files.

## Running as a non-root user
The .NET Linux container images include a new non-root user named `app` with the UID 1654. The UID is provided in an environment variable, `$APP_UID`. You can opt into this new user by adding the line `USER $APP_UID` to your Dockerfile.

The `USER $APP_UID` instruction in the Dockerfile ensures that the application runs under a non-root user inside the container. This is an important security best practice, as running processes as the root user can pose significant security risks.