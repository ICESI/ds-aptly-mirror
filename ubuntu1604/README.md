## Comandos instalación de mirror con aplty

## Equipo Mirror

### Generar la llave
```
gpg --gen-key
```
### En otra sesión generar la entropía
```
cat /dev/urandom
```
### Importar keyring
```
gpg --no-default-keyring --keyring /usr/share/keyrings/ubuntu-archive-keyring.gpg --export | gpg --no-default-keyring --keyring trustedkeys.gpg --import
```
### Seleccionar los paquetes a instalar
```
aptly mirror create -architectures=amd64 -filter='Priority (required) | Priority (important) | Priority (standard) | python3' -filter-with-deps xenial-main-python3 http://mirror.upb.edu.co/ubuntu/ xenial main
```
### Actualizar el mirror
```
aptly mirror update xenial-main-python3
```
### Crear un snaphost
```
aptly snapshot create xenial-snapshot-python3 from mirror xenial-main-python3
```
### Publicar el snapshot e ingresar una passphrase
```
aptly publish snapshot xenial-snapshot-python3
```
### Exportar la llave 
```
gpg --export --armor > my_key.pub
```
### Copiarla por fuera del equipo mirror desde el equipo anfitrión
```
scp vagrant@ip_solo_anfitrion:/tmp/my_key.pub .
```
### Iniciar el mirror
```
aptly serve
```
## Equipo Anfitrión
### Crear una sub-interface
```
sudo ip addr add 192.168.131.200/24 dev enp5s0:0
```
## Equipo Cliente
### Iniciar contenedor de docker
```
docker run -it --rm -v $(pwd)/keys:/tmp ubuntu:16.04 /bin/bash
```
### Configurar archivos en el contenedor para apuntar al mirror
```
cd /tmp
apt-key add my_key.pub
cd /etc/apt/
mv sources.list sources.list.backup
echo "deb http://192.168.131.100:8080/ xenial main" > sources.list
chmod 777 /tmp
apt-get clean
apt-get update
apt-get install python3
```
## Combinar Snapshots
### Cree un nuevo mirror de nombre xenial-main-ruby
### cree un snapshot de nombre xenial-snapshot-ruby
### Mezcle los snapshot
```
aptly snapshot merge -latest xenial-snapshot-python3 xenial-snapshot-ruby
aptly publish list
aptly publish drop xenial
```
