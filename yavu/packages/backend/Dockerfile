# --- ETAPA 1: Builder ---
FROM node:20-bookworm-slim AS builder

RUN apt-get update && apt-get install -y --no-install-recommends python3 g++ build-essential git && rm -rf /var/lib/apt/lists/*

WORKDIR /app

RUN corepack enable
COPY . .
RUN yarn install --immutable
RUN yarn build:all


# --- ETAPA 2: Runner ---
FROM node:20-bookworm-slim

WORKDIR /app

RUN corepack enable
RUN addgroup --system nonroot && adduser --system --ingroup nonroot nonroot

# --- CORRECCIÓN DE PERMISOS AQUÍ ---
# Nos aseguramos de que 'nonroot' sea el dueño de TODOS los archivos copiados.
COPY --from=builder --chown=nonroot:nonroot /app/yarn.lock /app/package.json ./
COPY --from=builder --chown=nonroot:nonroot /app/packages packages
COPY --from=builder --chown=nonroot:nonroot /app/plugins plugins
COPY --from=builder --chown=nonroot:nonroot /app/app-config*.yaml ./

RUN yarn workspaces focus --all --production

COPY --from=builder --chown=nonroot:nonroot /app/packages/backend/dist ./packages/backend/dist
COPY --from=builder --chown=nonroot:nonroot /app/packages/app/dist ./packages/app/dist

USER nonroot

EXPOSE 7007
CMD ["node", "packages/backend/dist/main.js"]
