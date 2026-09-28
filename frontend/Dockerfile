FROM node:24-bookworm-slim@sha256:0e0ff40c39bc087845bfb27465a0df4ea419520094bc35842ff83dd8cbe6f9b6

RUN mkdir -p /workspace/frontend/node_modules \
    && chown -R node:node /workspace/frontend
USER node
WORKDIR /workspace/frontend
EXPOSE 5173
CMD ["sh", "-c", "npm ci --no-fund --no-audit && exec npm run dev"]
