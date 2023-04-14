# tacos/Dockerfile
FROM nginx

COPY default.conf /etc/nginx/conf.d/
COPY html /usr/share/nginx/html/tacos/

CMD ["nginx", "-g", "daemon off;"]