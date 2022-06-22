# Description

This role deploys an instance of [NextCloud](https://nextcloud.com/) which is an Open Source self-hosted productivity platform.

# Configuration

The bare minimum would include:
```yaml
nextcloud_domain: 'cloud.example.org'
nextcloud_admin_username: 'admin'
nextcloud_admin_password: 'super-secret-password'
nextcloud_docs_secret_key: 'super-secret-key'
nextcloud_password_salt: 'super-secret-password-salt'
nextcloud_secret: 'super-secret-encryption-key'
```
There's also optional SMTP configuration:
```yaml
nextcloud_smtp_enabled: true
nextcloud_smtp_port: 587
nextcloud_smtp_host: 'smtp.example.org'
nextcloud_smtp_user: 'smtp-login-user'
nextcloud_smtp_pass: 'super-secret-password'
nextcloud_smtp_method: 'tls'
nextcloud_smtp_domain: 'example.org'
nextcloud_smtp_from: 'nextcloud'
```

# Management

The setup is created using [Docker Compose](https://docs.docker.com/compose/) and can be managed as such:
```
admin@node-01.do-ams3.nextcloud.misc:/docker/nextcloud % docker-compose ps
     Name                    Command                  State               Ports         
----------------------------------------------------------------------------------------
nextcloud-app     /entrypoint.sh apache2-for ...   Up (healthy)   0.0.0.0:9000->80/tcp  
nextcloud-cache   redis-server --requirepass ...   Up (healthy)   0.0.0.0:6379->6379/tcp
nextcloud-db      docker-entrypoint.sh postgres    Up (healthy)   0.0.0.0:5432->5432/tcp
nextcloud-docs    /bin/sh -c bash start-coll ...   Up (healthy)   0.0.0.0:9980->9980/tcp
```

# Backups

There's two folders that need to be backed up:

* `/docker/nextcloud/app/data` - Files created and uploaded by NextCloud users.
* `/docker/nextcloud/db/backup` - NextCloud PostgreSQL [database dump](https://www.postgresql.org/docs/13/app-pgdump.html)

The `data` folder contains especially important [`files_encryption`](https://docs.nextcloud.com/server/22/admin_manual/configuration_files/encryption_configuration.html) folders without which decryption of user data is impossible.

The database dumps are done with a [systemd timer](https://www.freedesktop.org/software/systemd/man/systemd.timer.html) and then backed up with [Restic](https://github.com/status-im/infra-role-restic-backups), as are the `data` folders:
```
admin@node-01.do-ams3.nextcloud.misc:~ % sudo systemctl list-timers -a '*-nextcloud-*.timer'
NEXT                        LEFT     LAST PASSED UNIT                        ACTIVATES                    
Fri 2021-08-06 00:00:00 UTC 11h left n/a  n/a    backup-nextcloud-data.timer backup-nextcloud-data.service
Fri 2021-08-06 00:00:00 UTC 11h left n/a  n/a    backup-nextcloud-db.timer   backup-nextcloud-db.service  
Fri 2021-08-06 00:00:00 UTC 11h left n/a  n/a    dump-nextcloud-db.timer     dump-nextcloud-db.service    

3 timers listed.
```

# Details

For more information on enryption see [`ENCRYPTION.md`](./ENCRYPTION.md).
