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

# Backup & Restore

See [BACKUP.md](./BACKUP.md) doc.

# Details

For more information on enryption see [`ENCRYPTION.md`](./ENCRYPTION.md).
