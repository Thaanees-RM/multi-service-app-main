# Stage 1: Build stage
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files first for caching
COPY package*.json ./

# Install all dependencies including devDependencies
RUN npm ci

# Copy source files
COPY . .

# Build production-ready static files using npx to run vite locally
RUN npm run build

# Stage 2: Production stage using nginx
FROM nginx:alpine

# Copy built files from builder stage to nginx public folder
COPY --from=builder /app/dist /usr/share/nginx/html

# Copy custom nginx config if you have one
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Expose port 80 for HTTP
EXPOSE 80

# Run nginx in foreground
CMD ["nginx", "-g", "daemon off;"]
