
# Golang Calculator Application with Multi-Stage Docker Build

This repository contains Dockerfiles for a Golang calculator application, demonstrating the use of multi-stage builds to create lean and efficient Docker images. This `README` will cover the basics of multi-stage builds, the benefits they offer, details on distroless images, and why we use `ubuntu` as a base image in the build stage.

---

## Dockerfile Overview

This project uses two Dockerfiles to highlight the difference between a standard build and a multi-stage build:

1. **Standard Build (One-Stage)**: Builds the Go application in a single stage.
2. **Multi-Stage Build**: Separates the build and runtime environments, producing a smaller, more secure image.

---

## Multi-Stage Docker Build

### What is a Multi-Stage Build?

A multi-stage build allows you to use multiple `FROM` statements in a single Dockerfile, which creates multiple stages within the build. Each stage can use a different base image and can pass only the essential files to the next stage, resulting in a lightweight final image.

### Benefits of Multi-Stage Builds

1. **Smaller Image Size**: Multi-stage builds significantly reduce the final image size by excluding unnecessary build dependencies.
2. **Improved Security**: Only the essential files and binaries are copied into the final image, reducing the attack surface.
3. **Optimization**: Each stage can be tailored with specific packages, tools, or configurations. Only essential runtime files end up in the final image.
4. **Easier Caching and Layer Management**: Docker caches each layer of the build process, so only updated layers need to be rebuilt, speeding up build times.

### Example: Multi-Stage Build Dockerfile

The `Dockerfile` provided demonstrates a multi-stage build for a Go application. Below is an explanation of each part:

```dockerfile
###########################################
# BASE IMAGE
###########################################

FROM ubuntu AS build

RUN apt-get update && apt-get install -y golang-go

ENV GO111MODULE=off

COPY . .

RUN CGO_ENABLED=0 go build -o /app .
```

1. **Build Stage**: The `ubuntu` image is used to set up the build environment with `golang-go`. 
2. **Build the Application**: We compile the application into a single binary file (`/app`), which will be copied into a new stage.

```dockerfile
############################################
# HERE STARTS THE MAGIC OF MULTI STAGE BUILD
############################################

FROM scratch

# Copy the compiled binary from the build stage
COPY --from=build /app /app

# Set the entrypoint for the container to run the binary
ENTRYPOINT ["/app"]
```

3. **Final Stage**: Using `FROM scratch`, we create a minimal final image, including only the compiled binary (`/app`). This eliminates the need for a full operating system, reducing image size and improving security.

---

## Distroless Images

**Distroless images** are minimal images that contain only the application and its runtime dependencies, without any package manager, shell, or OS utilities. The `scratch` base image is a form of a distroless image that essentially starts with nothing and is ideal for lightweight, secure containers.

### Why Use Distroless Images?

- **Security**: Fewer components mean fewer potential vulnerabilities.
- **Size Efficiency**: Since distroless images only include essential runtime files, they are often very small and quick to deploy.
- **Simplicity**: The lack of extraneous OS components means fewer things to maintain or troubleshoot.

---

## Why `Ubuntu` as a Base Image?

In a multi-stage build, using a standard OS like `ubuntu` in the build stage provides several advantages:

1. **Customization**: `ubuntu` gives us control over the tools, libraries, and configurations we add to the build environment.
2. **Dependency Management**: We can install any dependencies needed to build the application, regardless of the language or framework.
3. **Flexibility**: Using a full OS base for the build stage allows us to use a variety of compilers, debuggers, and other tools, making it a versatile choice for complex builds.

In the final stage, we switch to `scratch`, avoiding the unnecessary OS components and tools used in the build stage.

---

## Image Size Reduction with Multi-Stage Builds

By using a multi-stage build, we reduce the final image size considerably, as shown below:

- **Build Stage Image Size**: Includes OS dependencies, compilers, and all necessary tools (typically much larger).
- **Final Stage Image Size**: Contains only the binary and minimal dependencies, reducing the overall image size and load time.

For this Go application, the multi-stage approach results in an image containing only the compiled Go binary, making it compact and efficient.

---

## How to Build and Run the Docker Image

To build and run the multi-stage Docker image, follow these commands:

1. **Build the Image**:
   ```bash
   docker build -t golang-calculator .
   ```

2. **Run the Container**:
   ```bash
   docker run golang-calculator
   ```

This setup will run the compiled binary in a minimal runtime environment, leveraging the benefits of the multi-stage build process.

---

## Conclusion

Using multi-stage builds in Docker provides a streamlined, secure, and size-efficient approach to containerizing applications. This setup is ideal for production environments, where smaller images reduce deployment time, improve security, and allow for faster scaling.

---

### Additional Resources

- [Docker Multi-Stage Builds Documentation](https://docs.docker.com/develop/develop-images/multistage-build/)
- [Distroless Container Images](https://github.com/GoogleContainerTools/distroless)
- [Go Language](https://golang.org/)

