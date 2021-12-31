Callepuzzle Nextcloud
================================
Playbook para instalar Nextcloud en Debian 10 usando MySql Nextcloud y Nginx

Provision
---------
Se realiza mediante Ansible

### Dependencies
El playbook tiene como dependencias un rol y una colección:
* https://galaxy.ansible.com/jilgue/ansible_role_docker_nextcloud
* https://galaxy.ansible.com/nginxinc/nginx_core

Para instalarlas:
```bash
$ ansible-galaxy install --roles-path roles -r requirements.yml
$ ansible-galaxy collection install nginxinc.nginx_core
$ ansible-galaxy collection install community.general
```

### Instalación
```bash
$ ansible-playbook -i inventory.ini --vault-password-file .vault-password-file provision.yml
```

SSL
---
La gestión del certificado SSL se hace mediante [letsencrypt](https://letsencrypt.org) usando [certbot](https://certbot.eff.org/)
```
# certbot certonly --standalone -d cloud.callepuzzle.com
# cat /etc/letsencrypt/live/cloud.callepuzzle.com/cert.pem /etc/letsencrypt/live/cloud.callepuzzle.com/privkey.pem > /usr/local/etc/haproxy/cloud.callepuzzle.com.pem
```

Bugs
----

* https://help.nextcloud.com/t/connection-wizard-is-looping-between-log-in-and-grant-access/46809

```
That could be the reverse proxy. I had the same problem. The solution for me was to add the ‘overwriteprotocol’ variable to config.php and set it to “https”. See: https://github.com/nextcloud/server/blob/master/config/config.sample.php#L456-L463

Unfortunately, I ran directly into the next problem with it:

  'overwriteprotocol' => 'https',
```

* No arranca contenedor:

```
fatal: [callepuzzle]: FAILED! => {"changed": false, "msg": "Error starting container c9a681f5e5462fe207be7cddcf6794ad5aaebc28fef8286fce9cddbf0de19e26: 500 Server Error: Internal Server Error (\"Cannot link to a non running container: /nextcloud AS /haproxy/nextcloud\")"}
```

Mirar si exite el contenedor pero está parado, borrarlo y que el servicio cree uno nuevo.

Backup
------
```bash
$ docker exec -u www-data -it nextcloud php occ maintenance:mode --on
$ rsync -Aavx /srv/docker/nextcloud/nextcloud/data /backups/nextcloud-dirbkp_`date +"%Y%m%d"`/
$ docker exec -it mysql mysqldump --single-transaction -unextcloud -ppassword nextcloud_db > /srv/docker/nextcloud/nextcloud/data/backup.sql
$ docker exec -u www-data -it nextcloud php occ maintenance:mode --off
```

Actualizar Nextcloud
--------------------
Poner en modo mantenimiento:
```bash
$ docker exec -u www-data -it nextcloud php occ maintenance:mode --on
```

Cambiar la versión en la variable `nextcloud_version` y ejecutar:
```bash
$ ansible-playbook -i inventory.ini --vault-password-file .vault-password-file provision.yml --tags run-nextcloud
```

Quitar el modo mantenimiento:
```bash
$ docker exec -u www-data -it nextcloud php occ maintenance:mode --off
```


## TODO

apt install snapd
snap install core
snap refresh core
snap install --classic certbot
ln -s /snap/bin/certbot /usr/bin/certbot
certbot --nginx --register-unsafely-without-email


add /etc/nginx/dhparam

ansible-vault encrypt_string --vault-password-file .vault-password-file password --name nextcloud_db_pass
