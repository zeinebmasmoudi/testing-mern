# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build  # Creates optimized production build (in /dist or /build)

# Stage 2: Run (using Nginx for production)
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html  
EXPOSE 5177
CMD ["nginx", "-g", "daemon off;"]