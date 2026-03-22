# Example Dockerfile for developer's app
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --production
COPY . .

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app .
EXPOSE 8080
USER node
CMD ["node", "src/index.js"]
