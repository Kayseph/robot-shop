FROM node:20-bookworm-slim

EXPOSE 8080

WORKDIR /opt/server

COPY package.json /opt/server/
RUN npm install --omit=dev

COPY server.js tracing.js /opt/server/

CMD ["node", "-r", "./tracing.js", "server.js"]
