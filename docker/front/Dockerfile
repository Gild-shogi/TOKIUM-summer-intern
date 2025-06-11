FROM node:20-alpine

WORKDIR /app

RUN apk add curl
RUN corepack  enable

RUN mkdir /app/node_modules

RUN chown -R node:node /app/node_modules

USER node

CMD ["pnpm", "dev"]

EXPOSE 8080
