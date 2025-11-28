
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build -- --output-path=dist --configuration=production


FROM nginx:stable-alpine
COPY --from=builder /app/dist /usr/share/nginx/html


EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

