# 🛡️ Dockerfile Security & Best Practices Guide

## Table of Contents
- [Why Security-Focused Dockerfiles Matter](#why-security-focused-dockerfiles-matter)
- [Common Security Risks](#common-security-risks)
- [Best Practices](#best-practices)
- [Multi-Stage Build Deep Dive](#multi-stage-build-deep-dive)
- [Complete Examples](#complete-examples)
- [Security Scanning Tools](#security-scanning-tools)
- [Additional Security Considerations](#additional-security-considerations)

---

## Why Security-Focused Dockerfiles Matter

### 🔄 Containers Share the Host Kernel

Unlike virtual machines that run their own operating system, Docker containers share the same Linux kernel as the host system. This architectural difference creates unique security considerations:

- **Kernel-level vulnerabilities** can affect all containers on the host
- **Container escape attacks** can potentially access the host system
- **Root access within a container** translates to significant privileges on the host kernel

### 📦 Smaller Images = Smaller Attack Surface

The principle of minimalism applies strongly to container security:

- **Large base images** (like `ubuntu:latest`) include hundreds of packages you don't need
- **Each additional package** represents potential vulnerabilities
- **Unused services and tools** provide attack vectors for malicious actors
- **Minimal images** reduce both security risks and resource consumption

### 🚀 Optimized Production Builds

Development and production environments have different requirements:

- **Development tools** (compilers, debuggers, testing frameworks) shouldn't exist in production
- **Source code and build artifacts** may contain sensitive information
- **Package managers and build tools** can be exploited if present in runtime containers
- **Clean separation** between build and runtime environments is crucial

---

## Common Security Risks

### ❌ Critical Security Anti-Patterns

#### 1. **Large, Vulnerable Base Images**
```dockerfile
# AVOID: Large images with many packages
FROM ubuntu:20.04
FROM centos:8
```

#### 2. **Running as Root User**
```dockerfile
# AVOID: Default root execution
FROM node:18
COPY . .
CMD ["node", "app.js"]  # Runs as root by default
```

#### 3. **Hardcoded Secrets**
```dockerfile
# AVOID: Secrets in Dockerfile
ENV DATABASE_PASSWORD=supersecret123
ENV API_KEY=abc123xyz
```

#### 4. **Unnecessary Tools in Production**
```dockerfile
# AVOID: Build tools in runtime image
FROM node:18
RUN apt-get update && apt-get install -y \
    build-essential \
    python3 \
    curl \
    wget \
    vim
```

#### 5. **Poor Layer Management**
```dockerfile
# AVOID: Multiple layers and no cleanup
RUN apt-get update
RUN apt-get install -y nodejs
RUN apt-get install -y npm
RUN rm -rf /tmp/*  # Too late - layers already created
```

---

## Best Practices

### ✅ 1. Use Small & Trusted Base Images

#### Recommended Base Images:

| Language | Recommended Image | Size Comparison |
|----------|-------------------|-----------------|
| Node.js | `node:18-alpine` | ~40MB vs 300MB+ |
| Python | `python:3.11-slim` | ~50MB vs 200MB+ |
| Go | `golang:1.19-alpine` | ~100MB vs 400MB+ |
| Java | `openjdk:17-alpine` | ~80MB vs 250MB+ |

#### Why Alpine?
- **Minimal size**: Based on musl libc and BusyBox
- **Security focused**: Fewer packages = fewer vulnerabilities
- **Regular updates**: Active maintenance and security patches

```dockerfile
# Good: Small, official, regularly updated
FROM node:18-alpine
FROM python:3.11-slim
FROM golang:1.19-alpine
```

### ✅ 2. Implement Multi-Stage Builds

Multi-stage builds are the cornerstone of secure container images:

#### How Multi-Stage Works:
1. **Build Stage**: Contains all build tools, dependencies, source code
2. **Runtime Stage**: Contains only the compiled application and runtime dependencies
3. **Selective Copying**: Only necessary artifacts are copied to the final image

#### Benefits:
- **Reduced attack surface**: No build tools in production
- **Smaller images**: Faster deployments and less storage
- **Clean separation**: Build and runtime concerns are isolated

```dockerfile
# Stage 1: Build environment
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Stage 2: Production environment  
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
USER node
CMD ["node", "server.js"]
```

### ✅ 3. Run as Non-Root User

#### Why Non-Root Matters:
- **Container escape protection**: Limits damage if container is compromised
- **Principle of least privilege**: Application doesn't need root permissions
- **Defense in depth**: Additional security layer

#### Implementation Patterns:

```dockerfile
# Pattern 1: Create custom user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Pattern 2: Use existing non-root user (Node.js example)
USER node

# Pattern 3: Specify UID/GID for consistency
RUN adduser -D -s /bin/sh -u 1001 appuser
USER 1001
```

### ✅ 4. Secure Secret Management

#### Never Do This:
```dockerfile
# NEVER: Hardcoded secrets
ENV DB_PASSWORD=mysecret
ENV API_KEY=12345
```

#### Secure Alternatives:

```dockerfile
# 1. Runtime environment variables
# Pass at container start: docker run -e DB_PASSWORD=$DB_PASSWORD myapp

# 2. Docker secrets (Swarm mode)
# Mount secrets from orchestrator

# 3. External secret management
# Use HashiCorp Vault, AWS Secrets Manager, etc.
```

### ✅ 5. Optimize Layers and Cleanup

#### Efficient Layer Management:

```dockerfile
# Good: Combined commands with cleanup
RUN apk add --no-cache \
    nodejs \
    npm \
    && npm install -g yarn \
    && rm -rf /var/cache/apk/*

# Bad: Multiple layers, no cleanup
RUN apk update
RUN apk add nodejs
RUN apk add npm
RUN npm install -g yarn
```

#### Cleanup Best Practices:
- **Package manager caches**: `--no-cache` flag or manual cleanup
- **Temporary files**: Remove in the same RUN instruction
- **Build artifacts**: Don't copy unnecessary files

### ✅ 6. Implement Health Checks

```dockerfile
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1
```

---

## Multi-Stage Build Deep Dive

### Advanced Multi-Stage Pattern

```dockerfile
# ===== STAGE 1: Dependencies =====
FROM node:18 AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# ===== STAGE 2: Build =====
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# ===== STAGE 3: Runtime =====
FROM node:18-alpine AS runtime
WORKDIR /app

# Create non-root user
RUN addgroup -S nodejs && adduser -S nodejs -G nodejs

# Copy production dependencies
COPY --from=deps /app/node_modules ./node_modules

# Copy built application
COPY --from=builder /app/dist ./dist
COPY package*.json ./

# Security: Change ownership and switch user
RUN chown -R nodejs:nodejs /app
USER nodejs

# Application configuration
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

CMD ["node", "dist/server.js"]
```

### Stage-Specific Optimizations

#### Build Stage Optimizations:
```dockerfile
FROM node:18 AS builder
WORKDIR /app

# Leverage Docker layer caching
COPY package*.json ./
RUN npm ci

# Copy source code last (changes frequently)
COPY . .
RUN npm run build

# Optional: Remove non-production artifacts
RUN rm -rf node_modules && npm ci --only=production
```

#### Runtime Stage Optimizations:
```dockerfile
FROM node:18-alpine AS runtime

# Install security updates
RUN apk --no-cache upgrade

# Create dedicated user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Set up application directory
WORKDIR /app
COPY --from=builder /app/dist ./
COPY --from=builder /app/node_modules ./node_modules

# Security: Drop privileges
USER appuser

# Configure application
EXPOSE 3000
CMD ["node", "server.js"]
```

---

## Complete Examples

### 🟢 Node.js Application (Secure)

```dockerfile
# ===== Multi-stage Node.js application =====

# Stage 1: Build environment
FROM node:18 AS builder
WORKDIR /app

# Copy dependency manifests (for layer caching)
COPY package*.json ./
COPY yarn.lock ./

# Install all dependencies (including dev dependencies)
RUN yarn install --frozen-lockfile

# Copy source code and build
COPY . .
RUN yarn build

# Remove dev dependencies
RUN yarn install --production --frozen-lockfile

# Stage 2: Production environment
FROM node:18-alpine AS production
WORKDIR /app

# Install security updates
RUN apk --no-cache upgrade

# Create application user
RUN addgroup -S nodejs && adduser -S nodejs -G nodejs

# Copy built application and production dependencies
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/package*.json ./

# Switch to non-root user
USER nodejs

# Configure application
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

# Start application
CMD ["node", "dist/server.js"]
```

### 🟢 Python Application (Secure)

```dockerfile
# ===== Multi-stage Python application =====

# Stage 1: Build environment
FROM python:3.11 AS builder
WORKDIR /app

# Install system dependencies for building
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements and install Python dependencies
COPY requirements.txt .
RUN pip install --user --no-cache-dir --no-warn-script-location \
    -r requirements.txt

# Stage 2: Production environment
FROM python:3.11-slim AS production
WORKDIR /app

# Install security updates
RUN apt-get update && apt-get upgrade -y && rm -rf /var/lib/apt/lists/*

# Create application user
RUN useradd --create-home --shell /bin/bash python

# Copy Python packages from builder stage
COPY --from=builder --chown=python:python /root/.local /home/python/.local

# Copy application code
COPY --chown=python:python . .

# Update PATH to include user packages
ENV PATH=/home/python/.local/bin:$PATH

# Switch to non-root user
USER python

# Configure application
EXPOSE 8000
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD python -c "import requests; requests.get('http://localhost:8000/health')"

# Start application
CMD ["python", "app.py"]
```

### 🟢 Go Application (Secure)

```dockerfile
# ===== Multi-stage Go application =====

# Stage 1: Build environment
FROM golang:1.19 AS builder
WORKDIR /src

# Copy go mod files (for dependency caching)
COPY go.mod go.sum ./
RUN go mod download

# Copy source code and build
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

# Stage 2: Production environment
FROM alpine:3.17 AS production

# Install security updates and ca-certificates
RUN apk --no-cache add ca-certificates tzdata && \
    apk --no-cache upgrade

# Create non-root user
RUN adduser -D -s /bin/sh appuser

# Copy binary from builder stage
COPY --from=builder --chown=appuser:appuser /src/app /app

# Switch to non-root user
USER appuser

# Configure application
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/health || exit 1

# Start application
CMD ["/app"]
```

---

## Security Scanning Tools

### 🔍 Docker Desktop (Built-in)

```bash
# Scan image for vulnerabilities
docker scan myapp:latest

# Scan with specific severity
docker scan --severity high myapp:latest
```

### 🔍 Trivy (Recommended)

```bash
# Install Trivy
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin

# Scan Docker image
trivy image myapp:latest

# Scan with specific severities
trivy image --severity HIGH,CRITICAL myapp:latest

# Generate report in different formats
trivy image --format json --output report.json myapp:latest
```

### 🔍 Clair (Red Hat)

```bash
# Using Clair with Docker
docker run -d --name clair \
  -p 6060:6060 \
  quay.io/coreos/clair:latest
```

### 🔍 Snyk

```bash
# Install and authenticate Snyk
npm install -g snyk
snyk auth

# Scan Docker image
snyk container test myapp:latest
```

---

## Additional Security Considerations

### 🔒 Image Signing and Verification

```bash
# Sign images with Docker Content Trust
export DOCKER_CONTENT_TRUST=1
docker push myregistry/myapp:latest

# Verify signed images
docker pull myregistry/myapp:latest
```

### 🔒 Runtime Security

```dockerfile
# Drop unnecessary capabilities
# Run with: docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myapp

# Read-only filesystem
# Run with: docker run --read-only myapp

# No new privileges
# Run with: docker run --security-opt=no-new-privileges myapp
```

### 🔒 Network Security

```dockerfile
# Expose only necessary ports
EXPOSE 3000

# Consider using custom networks
# docker network create --driver bridge mynetwork
# docker run --network mynetwork myapp
```

### 🔒 Resource Limits

```bash
# Limit memory and CPU
docker run --memory=512m --cpus=1.0 myapp:latest

# In Docker Compose
version: '3.8'
services:
  app:
    image: myapp:latest
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '1.0'
```

---

## Security Checklist

### ✅ Pre-Build Checklist
- [ ] Use official, minimal base images
- [ ] Plan multi-stage build architecture
- [ ] Identify runtime dependencies vs build dependencies
- [ ] Review code for hardcoded secrets

### ✅ Dockerfile Checklist
- [ ] Multi-stage build implemented
- [ ] Non-root user created and used
- [ ] Minimal base image (Alpine/slim variants)
- [ ] Combined RUN commands with cleanup
- [ ] No secrets in environment variables or layers
- [ ] Health check implemented
- [ ] Proper port exposure

### ✅ Post-Build Checklist
- [ ] Image scanned for vulnerabilities
- [ ] Image size optimized (compare to base image)
- [ ] Security testing performed
- [ ] Documentation updated

### ✅ Runtime Checklist
- [ ] Container runs as non-root
- [ ] Resource limits configured
- [ ] Network security configured
- [ ] Secrets management implemented
- [ ] Monitoring and logging configured

---

## Conclusion

Implementing these Docker security best practices will result in:

- **🛡️ Reduced attack surface** through minimal images and multi-stage builds
- **🔒 Enhanced security** with non-root users and proper secret management  
- **🚀 Better performance** from smaller, optimized images
- **📊 Improved maintainability** with clear separation of concerns
- **🔍 Proactive security** through regular vulnerability scanning

Remember: Security is not a one-time setup but an ongoing process. Regularly update base images, scan for vulnerabilities, and stay informed about security best practices in the container ecosystem.

---

*Last updated: September 2025*