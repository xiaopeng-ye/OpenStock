# Fork-only build recipe — https://github.com/xiaopeng-ye/OpenStock
#
# Upstream (Open-Dev-Society/OpenStock) ships a Dockerfile but publishes no image, and its
# build needs a *reachable* MongoDB: lib/better-auth/auth.ts awaits getAuth() at module
# scope, which calls connectToDatabase() and throws when the URI is missing or the server
# cannot be selected. That is why upstream's README starts mongodb before `--build`.
#
# The BUILD_* args below are throwaway values supplied by the CI job (which runs a mongo:7
# service container). They are scoped to a single RUN, are never promoted to ENV, and never
# end up in the image's runtime environment — the container is configured entirely by
# Docker Compose at runtime. Never pass a real secret here.
#
# Build:
#   docker build -f docker/Dockerfile \
#     --add-host=host.docker.internal:host-gateway \
#     --build-arg BUILD_MONGODB_URI='mongodb://root:buildonly@host.docker.internal:27017/openstock?authSource=admin' \
#     -t ghcr.io/xiaopeng-ye/openstock:latest .

FROM node:20-alpine

WORKDIR /app
ENV NEXT_TELEMETRY_DISABLED=1

COPY package.json package-lock.json ./
RUN npm ci --no-audit --no-fund

COPY . .

ARG BUILD_MONGODB_URI
ARG BUILD_BETTER_AUTH_SECRET=build-only-placeholder-not-a-real-secret
ARG BUILD_BETTER_AUTH_URL=http://localhost:3000

# NEXT_PUBLIC_* values are inlined into the client bundle here and can never be changed at
# runtime, so anything of that kind has to be decided at image build time.
RUN MONGODB_URI="$BUILD_MONGODB_URI" \
    BETTER_AUTH_SECRET="$BUILD_BETTER_AUTH_SECRET" \
    BETTER_AUTH_URL="$BUILD_BETTER_AUTH_URL" \
    npm run build

EXPOSE 3000
CMD ["npm", "start"]
