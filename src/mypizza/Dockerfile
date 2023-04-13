# mypizza/Dockerfile
FROM nginx

COPY index.html /usr/local/apache2/htdocs/
COPY nginx-mypizza.conf /etc/nginx/conf.d/default.conf

CMD ["nginx", "-g", "daemon off;"]%