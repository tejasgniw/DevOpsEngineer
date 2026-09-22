# Containers

Containers isolate processes using namespaces and cgroups, sharing the host kernel, which makes them lightweight compared to traditional virtual machines

## Image Layers, Multi-stage build

- Docker images are layered, Each instruction in Dockerfile creates a layer

- Cached layers speed up builds


### Advantages of multi-stage build:

- Smaller production image

- No build tools in runtime

```
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /app
COPY . .
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app/out .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

# Container Security (Dockerfile best practices)

- Run as non-root user

```
FROM node:18-alpine

RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app
COPY . .

RUN chown -R appuser:appgroup /app

USER appuser

CMD ["node", "server.js"]

```

- Use minimal base images (alpine, distroless): I use minimal base images like Alpine or Distroless to reduce attack surface and CVE exposure.

- Image scanning (SAST, Trivy, etc.): All container images go through automated scanning for CVEs before being pushed to the registry

- Don’t bake secrets into images

- Use read-only filesystem

- Drop Linux capabilities


## Image tagging strategy

- Semantic versioning, COMMIT_SHA, semantic version + commit SHA

