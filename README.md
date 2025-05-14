# ---------- Stage 1: Build ----------
FROM node:22-slim as builder
ARG CI_COMMIT_TAG
ARG CI_COMMIT_SHORT_SHA

# Copy everything needed for build
COPY package.json package-lock.json tsconfig.app.json tsconfig.node.json tsconfig.json vite.config.ts svelte.config.js index.html ./
COPY src src/
COPY public public/

# Inject build version
RUN sed -i "s/COBBER_VERSION/${CI_COMMIT_TAG:-${CI_COMMIT_SHORT_SHA:-local}}/g" src/pages/Index.svelte

# Install & build
RUN npm ci
RUN npm run build


RUN cp public/appconfig.json dist/appconfig.json

# ---------- Stage 2: Serve ----------
FROM nginx:alpine


COPY --from=builder /dist /usr/share/nginx/html


RUN test -f /usr/share/nginx/html/appconfig.json

CMD ["nginx", "-g", "daemon off;"]
