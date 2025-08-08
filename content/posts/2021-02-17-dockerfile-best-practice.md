---
title:  "Dockerfile best practices"
date:   2021-02-17 19:57:00 +0100
---

This guide provides detailed explanations and recommendations for writing efficient, secure, and maintainable Dockerfiles.

## ARG

**Purpose:**  
Defines build-time variables that can be passed to the Docker build process using `--build-arg`.

**Best Practices:**  
- Use lower case for ARG variable names to distinguish them from ENV variables.
- Declare ARG variables close to where they are used in the Dockerfile for clarity.
- Avoid exposing sensitive information via ARG, as build arguments can be viewed in image history.

**Example:**
```dockerfile
ARG tag=latest
FROM alpine:$tag
```

## ENV

**Purpose:**  
Sets environment variables available at runtime inside the container.

**Best Practices:**  
- Use upper case for ENV variable names for consistency and visibility.
- Declare ENV variables near their usage to improve readability.
- Useful for updating system paths, configuration, or passing runtime options.

**Example:**
```dockerfile
ENV PATH="/usr/local/bin:${PATH}"
```

## ADD and COPY

**Purpose:**  
Adds files from your build context into the image.

**Best Practices:**  
- Prefer `COPY` over `ADD` unless you need features like remote URL fetching or archive extraction.
- Verify checksums of added files to ensure integrity.
- Copy files only once to the root directory, maintaining the directory structure as in the container.
- Use `.dockerignore` to exclude unnecessary files from the build context.

**Example:**
```dockerfile
COPY /bin/app /usr/local/bin/app
```

## ENTRYPOINT

**Purpose:**  
Defines the main command that always runs when the container starts.

**Best Practices:**  
- Use ENTRYPOINT for the primary executable, ensuring the container behaves as intended.

**Example:**
```dockerfile
ENTRYPOINT ["app"]
```

## CMD

**Purpose:**  
Provides default arguments to ENTRYPOINT or specifies the default command if ENTRYPOINT is not set.

**Best Practices:**  
- Use CMD for default flags or options, such as displaying help.
- Only one CMD instruction is allowed; later CMDs override earlier ones.

**Example:**
```dockerfile
CMD ["--help"]
```

## EXPOSE

**Purpose:**  
Documents which ports the container listens on at runtime.

**Best Practices:**  
- List all exposed ports in a single line for clarity.
- Add comments explaining the purpose of each port.

**Example:**
```dockerfile
EXPOSE 80 443 # 80: HTTP, 443: HTTPS
```

## RUN

**Purpose:**  
Executes commands during the build process, such as installing packages or configuring the image.

**Best Practices:**  
- Group related commands together to make refactoring easier.
- Place commands that are less likely to change earlier to maximize build cache efficiency.
- Use multi-line RUN instructions with `&&` to reduce image layers.
- Sort multi-line arguments for readability and maintainability.
- Use line continuations (`\`) for long lists.

**Example:**
```dockerfile
RUN apt-get update && \
    apt-get install -y \
    curl \
    git \
    vim && \
    rm -rf /var/lib/apt/lists/*
```

## USER

**Purpose:**  
Specifies the user to run the container process.

**Best Practices:**  
- Run containers as a non-root user for improved security.
- Make executables owned by root and not writable; non-root users only need execution permissions.

**Example:**
```dockerfile
RUN useradd -m appuser
USER appuser
```

## WORKDIR

**Purpose:**  
Sets the working directory for subsequent instructions.

**Best Practices:**  
- Avoid using `cd` inside RUN; use WORKDIR instead for clarity and maintainability.
- Set WORKDIR before copying files or running commands that depend on the directory.

**Example:**
```dockerfile
WORKDIR /app
COPY . .
```

## FROM

**Purpose:**  
Specifies the base image for the build.

**Best Practices:**  
- Use trusted base images (official, verified, or sponsored OSS) or build your own.
- Pin base images with digests instead of tags for reproducibility.
- Use multi-stage builds to separate build and runtime environments, reducing image size and attack surface.
- Create reusable build stages for shared artifacts.
- Use `scratch` or distroless images for minimal runtime environments.

**Example:**
```dockerfile
FROM node:18 AS build
WORKDIR /app
COPY . .
RUN npm install && npm run build

FROM node:18-slim AS runtime
WORKDIR /app
COPY --from=build /app/dist ./dist
CMD ["node", "dist/server.js"]
```

## VOLUME

**Purpose:**  
Declares mount points for persistent or shared data.

**Best Practices:**  
- Expose any files or directories created by the container that should be persisted or shared.
- Document the purpose of each volume.

**Example:**
```dockerfile
VOLUME /data
```

## .dockerignore

**Purpose:**  
Specifies files and directories to exclude from the build context.

**Best Practices:**  
- Always use `.dockerignore` to reduce build context size and prevent sensitive or unnecessary files from being added to the image.

**Example:**
```
node_modules
.git
*.log
```

## Secrets

**Best Practices:**  
- Never store secrets in the image or Dockerfile.
- Use environment variables, Docker secrets, or mount secrets at runtime.

## HEALTHCHECK

**Purpose:**  
Defines a command to check the health of the running container.

**Best Practices:**  
- Include a healthcheck whenever possible to monitor container status.
- Remember that the healthcheck command runs inside the container.

**Example:**
```dockerfile
HEALTHCHECK --interval=30s --timeout=5s \
  CMD curl -f http://localhost/health || exit 1
```

## Additional Tips

- Keep Dockerfiles readable and well-commented.
- Regularly update base images and dependencies to address security vulnerabilities.
- Test your images locally before deploying to production.
- Use automated builds and CI/CD pipelines for consistent image creation.
