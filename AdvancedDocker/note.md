## Multi - Staged Builds

```Dockerfile

# Build stage
FROM node:22-alpine AS builder

WORKDIR /app

# Install dependencies
COPY package.json package-lock.json* ./

RUN yarn install

# Copy all files and build
COPY . .

RUN yarn build

# Production stage
FROM node:22-alpine AS runner

WORKDIR /app

#Copy only nessary files from build stage
COPY --from=builder /app/public ./public
COPY --from=builder /app/.net ./.net
COPY --from=builder /app/package.json ./package.json
COPY --from=builder /app/node_modules ./node_modules

EXPOSE 3000

CMD ["yarn", "start"]
```

## Run Containers as non-root user

```Dockerfile
# Base image
FROM node:22-alpine

# Create non-root user and group
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app


# Copy Application file
COPY . .

# Ensure app files are owned by non-root user
RUN chown -R appuser:appgroup /app

# Swith to non-root user
USER appuser

EXPOSE 3000

CMD ["node", "server.js"]
```

## Resource limits (CPU,RAM)

```bash
docker run -d \
--name myapp \
--memory="512m" \
--memory-swarp="1g" \
--cpus="1.0" \
--pids-limit=100 \
-p 8080=3000 \
myapp:latest
```

## Vulnerability Scanning Tools

> Popular scanning tools;

```text
- Trivy (by Aqua Security)
- Docker scout
```

## How to scan Trivy tool

```bash
# Base image scan
trivy image myapp:latest

# Scan and show only HIGH and CRITICAL
trivy image --severity HIGH,CRITICAL myapp:latest

# Scan with ignore unfixed vulnerability
trivy image --ignore-unfixed myapp:latest

# Output image table format
trivy image -f table myapp:latest
```

## How to use Docker scout

```bash
# Enable docker scout (one-time)
docker scout quickview

# View detail report
docker scout cves myapp:latest
```

## Run locally .env file

```bash
docker run -d \
--env-file .env \
-p 3000:3000
--name myapp:latest
```

## Docker Healthcheck

```Dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package.json .

RUN npm install --production

HEALTHCHECK --interval=30s \
    --timeout=5s --retries=3 --start-period=10s \
    CMD curl --fail http://localhost=3000 || exit 1
```

## How to set up Restart Policy

```bash
docker run -d \
--restart unless-stopped \
--name myapp myapp:latest
```

## Docker compose

```yml
services:
  myapp:
    image: myapp:latest
    restart: unless-stopped
```

## Docker Buildx

> Build Multi-Platform image;

```bash
docker buildx build --platform linux/amd64,linux/arm64 -t dockerhubuser/myapp:1.0 --push .
```

## Verify image platform

```bash
docker buildx imagetools inspect dockerhubuser/myapp:1.0

Platforms:
    - Name: ---

```
