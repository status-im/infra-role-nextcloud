# Backups

There's two folders that need to be backed up:

* `/docker/nextcloud/app/data` - Files created and uploaded by NextCloud users.
* `/docker/nextcloud/db/backup` - NextCloud PostgreSQL [database dump](https://www.postgresql.org/docs/13/app-pgdump.html)

The `data` folder contains especially important [`files_encryption`](https://docs.nextcloud.com/server/22/admin_manual/configuration_files/encryption_configuration.html) folders without which decryption of user data is impossible.

The database dumps are done with a [systemd timer](https://www.freedesktop.org/software/systemd/man/systemd.timer.html) and then backed up with [Restic](https://github.com/status-im/infra-role-restic-backups), as are the `data` folders:
```
jakubgs@node-01.he-eu-hel1.office.cloud:~ % sudo systemctl list-timers -a '*-nextcloud-*.timer'
NEXT                        LEFT     LAST                        PASSED  UNIT                        ACTIVATES
Sat 2025-07-26 00:00:00 UTC 11h left Fri 2025-07-25 00:00:03 UTC 12h ago backup-nextcloud-data.timer backup-nextcloud-data.service
Sat 2025-07-26 00:00:00 UTC 11h left Fri 2025-07-25 00:00:03 UTC 12h ago backup-nextcloud-db.timer   backup-nextcloud-db.service
Sat 2025-07-26 00:00:00 UTC 11h left Fri 2025-07-25 00:00:03 UTC 12h ago dump-nextcloud-db.timer     dump-nextcloud-db.service
```

# Restoring

## Backup Existing Data

Create a copy of NextCloud folder:
```bash
cd /docker/nextcloud
docker compose down
cd ../
sudo cp -r nextcloud nextcloud_bkp
```

## Restore Backup

To restore backup to the same directory the destination permissions need to be changed:
```bash
sudo chmod g+w -R /docker/nextcloud/db/backup/nextcloud
sudo -i -u restic restic restore --target=/ 033ce34d
```
```
Summary: Restored 5 files/dirs (0 B) in 0:00, skipped 185 files/dirs 2.200 MiB
```
By using `--target=/` we restore the backup to the same location from which it was copied.

## Load Database Dump

:warning: This is a destructive operation, especially when done with `--clean`.

You can do it from the host:
```bash
source /docker/hedgedoc/.psql.env
sudo -E \
    pg_restore --clean -U nextcloud -d nextcloud /backup/nextcloud
```
Of from within the container:
```bash
docker exec -it nextcloud-db \
    pg_restore --clean -U nextcloud -d nextcloud /backup/nextcloud
```

If you only want to restore deleted records you can try `--data-only` flag.

The `nexcloud-app` container needs to be restarted afterewards.

## Known Issues

* `Social Login` plugin might require disabling and re-enabling to work.
