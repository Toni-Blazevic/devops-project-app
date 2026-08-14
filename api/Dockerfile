# Dependencies za production
FROM node:22-alpine AS dependencies

WORKDIR /app

COPY package*.json ./
RUN npm install --omit=dev


# Development
FROM node:22-alpine AS development

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY src ./src

CMD ["npm", "run", "dev"]


# Production runtime
FROM node:22-alpine AS runtime

WORKDIR /app

ENV NODE_ENV=production

COPY --from=dependencies --chown=node:node /app/node_modules ./node_modules
COPY --chown=node:node src ./src

RUN rm -rf /usr/local/lib/node_modules/npm \
    && rm -f /usr/local/bin/npm /usr/local/bin/npx

USER node

EXPOSE 8080

CMD ["node", "src/server.js"]