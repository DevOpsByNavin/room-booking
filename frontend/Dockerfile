FROM node:alpine AS builder

WORKDIR /app
COPY frontend/package.json ./
COPY frontend/package-lock.json ./
RUN npm ci 
COPY frontend .
RUN npm run build


FROM nginx:alpine

RUN rm /etc/nginx/conf.d/default.conf
COPY nginx/nginx.conf /etc/nginx/nginx.conf
COPY --from=builder /app/dist /var/www/room-booking
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]