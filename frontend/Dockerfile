# --- Stage 1: Build Angular ---
FROM node:18-alpine AS build

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

RUN npm run build --prod

# --- Stage 2: Nginx Server ---
FROM nginx:alpine

# Copy custom nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copy built Angular files
COPY --from=build /app/dist/angular-15-crud  /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]