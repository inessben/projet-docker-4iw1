# ── STAGE 1 : development ──────────────────────────────────
FROM node:20-alpine AS development

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

RUN addgroup -S appgroup && adduser -S appuser -G appgroup \
    && chown -R appuser:appgroup /app
USER appuser

EXPOSE 3001
CMD ["npx", "nodemon", "src/app.js"]


# ── STAGE 2 : production ───────────────────────────────────
FROM node:20-alpine AS production

ENV NODE_ENV=production

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

COPY src/ ./src/

RUN addgroup -S appgroup && adduser -S appuser -G appgroup \
    && chown -R appuser:appgroup /app
USER appuser

EXPOSE 3001

HEALTHCHECK --interval=30s --timeout=5s --start-period=15s \
    CMD node -e "require('http').get('http://localhost:3001/health', r => r.statusCode === 200 ? process.exit(0) : process.exit(1))"

CMD ["node", "src/app.js"]