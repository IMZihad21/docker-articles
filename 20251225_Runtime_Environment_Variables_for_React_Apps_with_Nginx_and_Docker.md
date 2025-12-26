React environment variables are normally injected at build time. Once the app is built, changing API URLs, feature flags, or monitoring config requires rebuilding the image. This becomes a problem when the same Docker image needs to run in staging, production, or other environments.

This setup allows runtime configuration for React apps served by Nginx in Docker. The image is built once and configured when the container starts.

## How it works

### Add `public/runtime-env.template.js`

```js
window.RUNTIME_ENV = {
  API_URL: "${API_URL}",
  FEATURE_FLAG_ANALYTICS: "${FEATURE_FLAG_ANALYTICS}",
  FEATURE_FLAG_NEW_DASHBOARD: "${FEATURE_FLAG_NEW_DASHBOARD}",
  SENTRY_DSN: "${SENTRY_DSN}",
};
```

### In `index.html`, add in `<head>`:

```html
<script src="/runtime-env.js"></script>
```

### In your code:

```js
const config = {
  API_URL: 'https://fallback.com',
  FEATURE_FLAG_ANALYTICS: false,
  // defaults
  ...(window.RUNTIME_ENV || {}),
};

export default config;
```

### Dockerfile

```dockerfile
# Build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

### Runtime
FROM nginx:alpine
RUN apk add --no-cache gettext
COPY --from=builder /app/build /usr/share/nginx/html  # /dist for Vite
COPY public/runtime-env.template.js /usr/share/nginx/html/runtime-env.template.js
COPY entrypoint.sh /docker-entrypoint.d/40-runtime-env.sh
RUN chmod +x /docker-entrypoint.d/40-runtime-env.sh
CMD ["nginx", "-g", "daemon off;"]
```

### `entrypoint.sh`

```bash
#!/bin/sh
set -eu

export API_URL=${API_URL:-}
export FEATURE_FLAG_ANALYTICS=${FEATURE_FLAG_ANALYTICS:-}
export FEATURE_FLAG_NEW_DASHBOARD=${FEATURE_FLAG_NEW_DASHBOARD:-}
export SENTRY_DSN=${SENTRY_DSN:-}

envsubst '${API_URL} ${FEATURE_FLAG_ANALYTICS} ${FEATURE_FLAG_NEW_DASHBOARD} ${SENTRY_DSN}' \
  < /usr/share/nginx/html/runtime-env.template.js \
  > /usr/share/nginx/html/runtime-env.js

rm /usr/share/nginx/html/runtime-env.template.js

exec "$@"
```

### Run

```bash
docker run -p 80:80 \
  -e API_URL=https://api.prod.com \
  -e FEATURE_FLAG_ANALYTICS=true \
  your-image:latest
```

Done. Container generates the config file on startup from env vars. Safe because we only allow specific ones with envsubst.

## Why this is useful

With this approach, the frontend behaves more like a backend service:

* One Docker image can be reused across environments
* API URLs, feature flags, and DSNs can change without rebuild
* Works well with Docker, Kubernetes, and CI/CD pipelines
* Configuration is explicit and controlled

It avoids the common issue where frontend config is tightly coupled to the build step.

## How it works

A small JavaScript template is added to the public folder. At container startup, Nginx runs a script that replaces placeholders in this file with actual environment variable values using `envsubst`. The generated file is then loaded before the React app and exposes the config through `window.RUNTIME_ENV`.

Only variables explicitly listed in the template are exposed, reducing the risk of leaking unintended values.

## Result

You get true runtime configuration for a static React app, without rebuilding the image. The same Docker image can safely run in different environments with different settings, using standard environment variables.
