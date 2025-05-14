# ---------- Stage 1: Build ----------
FROM node:22-slim as builder

ARG CI_COMMIT_TAG
ARG CI_COMMIT_SHORT_SHA

WORKDIR /app
COPY package.json package-lock.json tsconfig.app.json tsconfig.node.json tsconfig.json vite.config.ts svelte.config.js index.html ./
COPY src ./src
COPY public ./public

RUN sed -i "s/COBBER_VERSION/${CI_COMMIT_TAG:-${CI_COMMIT_SHORT_SHA:-local}}/g" src/pages/Index.svelte
RUN npm ci
RUN npm run build

# ---------- Stage 2: Serve ----------
FROM nginx:alpine
WORKDIR /usr/share/nginx/html

COPY --from=builder /app/dist .
COPY --from=builder /app/public/appconfig.json ./appconfig.json

CMD ["nginx", "-g", "daemon off;"]
