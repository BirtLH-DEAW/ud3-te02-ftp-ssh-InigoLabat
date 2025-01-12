# Utilizamos una imagen oficial de Ubuntu
FROM ubuntu:latest

# Damos información sobre la imagen que estamos creando
LABEL \
    version="1.0" \
    description="Ubuntu + Apache2 + virtual host + ProFTPD + ssh" \
    maintainer="usuarioBIRT <usuarioBIRT@birt.eus>"

# # Actualizamos la lista de paquetes e instalamos nano, apache2 y proftpd junto con sus modulos
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y nano apache2 proftpd proftpd-mod-crypto ssh git && \
    rm -rf /var/lib/apt/lists/*

# Creamos directorios para los sitios web y configuraciones
RUN mkdir -p /var/www/html/sitioprimero /var/www/html/sitiosegundo

# Copiamos archivos al contenedor
COPY /apache/indexPrimero.html /apache/indexSegundo.html /apache/sitioPrimero.conf /apache/sitioSegundo.conf /apache/sitioPrimero.key /apache/sitioPrimero.cer /

# Movemos los archivos a sus ubicaciones adecuadas
RUN mv /indexPrimero.html /var/www/html/sitioprimero/index.html && \
    mv /indexSegundo.html /var/www/html/sitiosegundo/index.html && \
    mv /sitioPrimero.conf /etc/apache2/sites-available/sitioPrimero.conf && \
    mv /sitioSegundo.conf /etc/apache2/sites-available/sitioSegundo.conf && \
    mv /sitioPrimero.key /etc/ssl/private/sitioPrimero.key && \
    mv /sitioPrimero.cer /etc/ssl/certs/sitioPrimero.cer

# Habilitamos los sitios y el módulo SSL
RUN a2ensite sitioPrimero.conf && \
    a2ensite sitioSegundo.conf && \
    a2enmod ssl

# # Copiamos los archivos necesarios para la configuracion del servidor ftp
COPY /ftp/proftpd.conf /ftp/tls.conf /ftp/proftpd.key /ftp/proftpd.crt /ftp/modules.conf /

# # Y los movemos a sus ubicaciones
RUN mv /proftpd.conf /etc/proftpd/proftpd.conf && \
    mv /tls.conf /etc/proftpd/tls.conf && \
    mv /proftpd.key /etc/ssl/private/proftpd.key && \
    mv /proftpd.crt /etc/ssl/certs/proftpd.crt && \
    mv /modules.conf /etc/proftpd/modules.conf

# # Copiamos los archivos necesarios para la configuracion del servidor ssh
COPY /ssh/sshd_config /ftp/ftpusers /

# # Y los movemos a sus ubicaciones
RUN mv /sshd_config /etc/ssh/sshd_config && \
    mv /ftpusers /etc/ftpusers

# # Creamos el usuario del ej1 y le asignamos una contraseña
RUN useradd -d /var/www/html/sitioprimero -s /bin/false inigo1 && \
    echo "inigo1:deaw" | chpasswd

# # Creamos el usuario del ej2 y le asignamos una contraseña
RUN useradd -d /var/www/html/sitiosegundo -s /bin/bash inigo2 && \
    echo "inigo2:deaw" | chpasswd
RUN chown inigo2 /var/www/html/sitiosegundo

# # Copiamos la clave privada para clonar el repositorio y le damos permisos
COPY /ssh/id_rsa /root/.ssh 
RUN chmod 600 /root/.ssh/id_rsa
RUN ssh-keyscan github.com >> /root/.ssh/known_hosts

# # Clonamos el repositorio mediante el enlace directamente a la carpeta configurada para los usuarios anonimos
RUN git clone git@github.com:deaw-birt/UD3-ftp_anonimo.git /srv/ftp/repositorio

# Exponemos los puertos
EXPOSE 80
EXPOSE 443
EXPOSE 1022
EXPOSE 21
EXPOSE 50000-50030
# Para el modo activo
EXPOSE 20

# Comando por defecto al iniciar el contenedor
#CMD ["apachectl", "-D", "FOREGROUND"]
CMD service apache2 start && service proftpd start && service ssh start && tail -f /dev/null