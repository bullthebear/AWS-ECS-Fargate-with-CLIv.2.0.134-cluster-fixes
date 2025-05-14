# ---------- Stage 1: Build ----------
FROM node:22-slim as builder
ARG CI_COMMIT_TAG
ARG CI_COMMIT_SHORT_SHA

# Copy and install
COPY package.json package-lock.json tsconfig.app.json tsconfig.node.json tsconfig.json vite.config.ts svelte.config.js index.html ./
COPY src src/
COPY public public/

RUN sed -i "s/COBBER_VERSION/${CI_COMMIT_TAG:-${CI_COMMIT_SHORT_SHA:-local}}/g" src/pages/Index.svelte

RUN npm ci
RUN npm run build


COPY public/appconfig.json dist/appconfig.json

# ---------- Stage 2: Serve ----------
FROM nginx:alpine
COPY --from=builder /dist /usr/share/nginx/html
CMD ["nginx", "-g", "daemon off;"]
