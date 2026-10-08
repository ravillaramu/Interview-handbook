# Dockerfile Examples

The examples use a Node.js app with a `package-lock.json`, a `build` script, and a production entry point at `dist/server.js`. Adjust paths, port, and commands for your application.

## Simple Dockerfile

This single-stage example is useful for learning and local development. It includes build-time dependencies and is not the preferred production image for most applications.

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

ENV NODE_ENV=production
EXPOSE 3000

CMD ["node", "dist/server.js"]
```

## Production-oriented multi-stage Dockerfile

The builder stage installs all dependencies and builds the app. The runtime stage contains only production dependencies and compiled output, and runs as the unprivileged `node` user.

```dockerfile
FROM node:22-alpine AS build

WORKDIR /app
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM node:22-alpine AS runtime

ENV NODE_ENV=production
WORKDIR /app

COPY package*.json ./
RUN npm ci --omit=dev \
    && npm cache clean --force

COPY --from=build --chown=node:node /app/dist ./dist

USER node
EXPOSE 3000

CMD ["node", "dist/server.js"]
```

For stronger supply-chain reproducibility, pin base images by reviewed digest and update them through a controlled process. Scan the final image in CI and rebuild regularly to pick up security updates. Do not pass credentials using Docker `ARG` or copy secrets into the build context; use BuildKit secret mounts for private build-time credentials.

## `.dockerignore`

```dockerignore
.git
.github
node_modules
npm-debug.log
coverage
dist
.env
.env.*
*.pem
*.key
```

Review ignore patterns against your build; for example, omit `dist` only if the Dockerfile builds it inside the image.

## Additional runtime hardening

At deployment time, where the app supports it:

- Use a read-only root filesystem and provide only the writable temporary volumes the process needs.
- Drop Linux capabilities, disallow privilege escalation, and set CPU/memory limits.
- Keep secrets out of image layers and environment defaults; inject them from a secret manager at runtime.
- Use an image scanner and deploy by immutable tag or digest.

Example local run with a read-only root filesystem and a writable `/tmp`:

```sh
docker run --rm \
  --read-only \
  --tmpfs /tmp:rw,noexec,nosuid,size=64m \
  --cap-drop ALL \
  --security-opt no-new-privileges \
  --memory 256m \
  --cpus 0.5 \
  -p 3000:3000 \
  sample-app:local
```

The process must already run as a non-root user inside the image and the application must not need other writable paths for these settings to work.