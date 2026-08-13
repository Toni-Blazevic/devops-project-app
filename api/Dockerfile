FROM node:22-alpine as dependencies

WORKDIR /app

COPY package*.json ./
RUN npm install --omit-dev


FROM node:22-alpine as runtime

WORKDIR /app

COPY --from=dependencies --chown=node:node /app/node_modules ./node_modules
COPY --chown=node:node package*.json ./
COPY --chown=node:node src ./src

EXPOSE 8080

USER node

CMD ["npm", "start"]