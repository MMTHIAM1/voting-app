FROM nginx:alpine

# Création du répertoire des certificats
RUN mkdir -p /etc/nginx/certs

# Copie du fichier de configuration nginx
COPY default.conf /etc/nginx/conf.d/default.conf

# Copie des certificats SSL
COPY certs/selfsigned.crt /etc/nginx/certs/selfsigned.crt
COPY certs/selfsigned.key /etc/nginx/certs/selfsigned.key
