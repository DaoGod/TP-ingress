# mypizza/Dockerfile
FROM nginx

COPY default.conf /etc/nginx/conf.d/
COPY html /usr/share/nginx/html/mypizza/

CMD ["nginx", "-g", "daemon off;"]