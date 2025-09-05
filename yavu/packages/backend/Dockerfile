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

# Creamos y nos movemos al directorio de la app
WORKDIR /app

# Habilitamos el gestor de paquetes de Yarn (necesario para el runtime)
RUN corepack enable

# Creamos un usuario no-root para seguridad
RUN addgroup --system nonroot && adduser --system --ingroup nonroot nonroot

# Copiamos solo los archivos de producción
COPY --from=builder /app/packages/backend/dist/bundle.tar.gz ./bundle.tar.gz
COPY --from=builder /app/app-config.yaml ./app-config.yaml
COPY --from=builder /app/app-config.production.yaml ./app-config.production.yaml
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/examples ./examples

# Descomprimimos el bundle en la raíz de la aplicación
RUN tar xzf bundle.tar.gz && rm bundle.tar.gz

# Aseguramos los permisos de archivos
RUN chown -R nonroot:nonroot .

# Cambiamos al usuario no-root
USER nonroot

# Exponemos el puerto del backend
EXPOSE 7007

# El comando de inicio ahora funcionará con la ruta correcta
CMD ["node", "./packages/backend/dist/index.cjs.js"]
