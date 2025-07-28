FROM node:20-alpine

WORKDIR /app

# Copy package.json files and install dependencies
COPY package*.json ./

RUN npm install

# Copy the rest of the source code
COPY . .

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

CMD ["node", "server.js"]
