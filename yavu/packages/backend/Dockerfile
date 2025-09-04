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

# Copiamos todos los archivos necesarios (su dueño inicial será root)
COPY --from=builder /app/yarn.lock /app/package.json ./
COPY --from=builder /app/packages packages
COPY --from=builder /app/plugins plugins
COPY --from=builder /app/app-config*.yaml ./

RUN yarn workspaces focus --all --production

COPY --from=builder /app/packages/backend/dist ./packages/backend/dist
COPY --from=builder /app/packages/app/dist ./packages/app/dist

# --- EL CAMBIO MÁGICO Y DEFINITIVO ---
# Cambiamos el dueño de TODA la carpeta /app y su contenido a 'nonroot'.
RUN chown -R nonroot:nonroot /app

# Ahora que 'nonroot' es dueño de todo, cambiamos de usuario.
USER nonroot

EXPOSE 7007
CMD ["node", "packages/backend/dist/main.js"]
