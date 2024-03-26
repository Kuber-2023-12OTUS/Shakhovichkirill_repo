FROM nginx:alpine

COPY nginx.conf /etc/nginx/

WORKDIR /app/

COPY app .

EXPOSE 8000

USER 1001
