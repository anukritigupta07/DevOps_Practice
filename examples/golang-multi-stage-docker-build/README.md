# Multi Stage Docker Build

The main purpose of choosing a golang based applciation to demostrate this example is golang is a statically-typed programming language that does not require a runtime in the traditional sense. Unlike dynamically-typed languages like Python, Ruby, and JavaScript, which rely on a runtime environment to execute their code, Go compiles directly to machine code, which can then be executed directly by the operating system.

So the real advantage of multi stage docker build and distro less images can be understand with a drastic decrease in the Image size.

# Day 26 — Multi-Stage

# Day 26 — Multi-Stage Docker Builds

## Overview

Day 26 focuses on **Multi-Stage Docker Builds** and **Distroless Container Images**. The goal is to understand how to reduce Docker image size by separating the application build process from the final runtime environment.

### Topics Covered

- Multi-stage Docker builds
- Traditional versus optimized container images
- Building Go applications inside Docker
- Using `AS` to name a build stage
- Using `COPY --from` to transfer build artifacts between stages
- Understanding distroless container images
- Reducing the final Docker image size

---

## 1. What Is a Multi-Stage Docker Build?

A multi-stage build uses multiple `FROM` instructions in a single Dockerfile.

Each `FROM` instruction begins a new build stage. We can use one stage to compile or build an application and another stage to run it.

The final image only needs the application and its runtime dependencies, rather than all the tools used during compilation.

### Basic Workflow

```text
Application Source Code
          |
          v
     Build Stage
          |
          v
  Compile Application
          |
          v
    Build Artifact
          |
          v
    Runtime Stage
          |
          v
    Final Image
          |
          v
   Running Container
```

### Benefits

- Reduces unnecessary files in the final image.
- Can significantly reduce image size.
- Separates build dependencies from runtime dependencies.
- Reduces the tools available inside the final container when a minimal runtime image is used.

---

## 2. Understanding the Build Stage

The instructor demonstrates a Go application build using a Dockerfile similar to the following.

```dockerfile
# BASE IMAGE
FROM ubi-minimal AS build

RUN apt-get update && apt-get install -y golang-go

ENV GO111MODULE=off

COPY . .

RUN CGO_ENABLED=0 go build -o /app .

ENTRYPOINT ["/app"]
```

**Note:** This reproduces the instructions visible in the lesson screenshot. The package-install command and filesystem setup may require adjustment for the chosen base image.

### Explanation

#### `FROM ubi-minimal AS build`

```dockerfile
FROM ubi-minimal AS build
```

- Selects the base image.
- Names the stage `build`.
- Allows later stages to refer to this stage by name.

#### `RUN`

```dockerfile
RUN apt-get update && apt-get install -y golang-go
```

Runs commands during image construction to install the Go compiler. The package manager must be compatible with the selected base image.

#### `ENV`

```dockerfile
ENV GO111MODULE=off
```

Sets an environment variable that controls Go module behavior for this build.

#### `COPY`

```dockerfile
COPY . .
```

Copies files from the Docker build context into the current working directory in the image.

#### `go build`

```dockerfile
RUN CGO_ENABLED=0 go build -o /app .
```

- `CGO_ENABLED=0` disables CGO for this build.
- `go build` compiles the Go application.
- `-o /app` specifies the output binary path.
- `.` identifies the package to build.

The destination directory must exist, and the source must be a valid Go project.

#### `ENTRYPOINT`

```dockerfile
ENTRYPOINT ["/app"]
```

Configures `/app` as the executable when the container starts. In this example, it is intended to be the compiled Go binary.

---

## 3. Copying Artifacts Between Stages

The most important instruction in a multi-stage Dockerfile is:

```dockerfile
COPY --from=build /app /app
```

This copies `/app` from the stage named `build` into the current stage.

A later stage can use a minimal runtime image and copy only the application binary it needs.

For example, the overall structure could be:

```dockerfile
FROM <go-build-image> AS build

WORKDIR /src
COPY . .
RUN CGO_ENABLED=0 go build -o /app .

FROM <minimal-runtime-image>
COPY --from=build /app /app
ENTRYPOINT ["/app"]
```

The placeholders must be replaced with compatible images. The build stage must also install Go and provide the correct source files and build configuration.

### Key Concept

The build stage contains the tools needed to produce the application. The final stage contains the artifact and the runtime requirements needed to execute it.

---

## 4. What Are Distroless Container Images?

Distroless images contain an application and its required runtime components without the usual collection of general-purpose operating-system utilities.

They generally do not include a shell, package manager, or common debugging tools.

### Why Use Distroless Images?

- Keep the runtime image minimal.
- Reduce unnecessary software in the container.
- Reduce the number of tools potentially available to an attacker.
- Support a smaller runtime footprint.

### Important Considerations

Distroless images are not automatically suitable for every application. The correct runtime libraries, certificates, permissions, and other required files must still be available.

Because a shell and common troubleshooting utilities are usually absent, debugging may require different techniques.

---

## 5. Multi-Stage Builds vs. Distroless Images

| Multi-stage build | Distroless image |
|---|---|
| A Dockerfile build technique | A type of minimal runtime image |
| Separates build and runtime stages | Omits most general-purpose OS utilities |
| Uses `FROM` for multiple stages | Used as the base image for a runtime stage |
| Transfers artifacts using `COPY --from` | Runs the application with its required runtime components |

These approaches can be combined to produce a smaller, more focused final container image.

---

## 6. Why Reduce Docker Image Size?

Large images can take longer to transfer, consume more storage, and increase the amount of software included in the runtime environment.

Multi-stage builds help by keeping build-only dependencies out of the final image.

The actual size reduction depends on the application, its dependencies, the selected base images, and the files copied into the final stage.

---

## 7. Important Docker Instructions

| Instruction | Purpose |
|---|---|
| `FROM` | Selects a base image and begins a stage |
| `AS` | Names a build stage |
| `RUN` | Executes commands during the build |
| `ENV` | Sets environment variables |
| `COPY` | Copies files into the image |
| `COPY --from` | Copies files from another build stage |
| `ENTRYPOINT` | Configures the container's main executable |

---

## 8. Key Takeaways

1. A multi-stage build uses multiple stages in one Dockerfile.
2. The build stage contains the tools required to compile or build the application.
3. `AS build` assigns a name to a stage.
4. `COPY --from=build` transfers artifacts from a previous stage.
5. Distroless images minimize the runtime environment by excluding most general-purpose OS utilities.
6. Multi-stage builds and distroless images can be combined.
7. A smaller final image can improve transfer times, storage efficiency, and runtime security posture.

---

## Final Workflow

```text
Source Code
    |
    v
Build Stage
    |
    v
Compile Application
    |
    v
Copy Required Artifact
    |
    v
Minimal Runtime Stage
    |
    v
Final Docker Image
    |
    v
Run Container
```

**Day 26 objective:** Understand how multi-stage Docker builds separate compilation from execution and how minimal runtime images can reduce the size and contents of the final container image.


