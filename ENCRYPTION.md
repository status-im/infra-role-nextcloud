# Description

NextCloud supports [server-side encryption](https://docs.nextcloud.com/server/latest/admin_manual/configuration_files/encryption_configuration.html) which is covers both local files and remote files stored on service like Dropbox or Google Drive.

# Details

There are four types of encryption available using four types of keys:

* __Master key__ - Default with one central key which can decrypto files of all users.
* __User key__ - Deprecated scheme. Could cause data loss if user los their password.
* __Recovery key__ - Recovery mechanism for user key scheme if password is lost.
* __Public sharing key__ - Used to secure files shared publicly, independent from master.

In our case we use the simplest and default scheme of using a single master key.

This key is defined in `config.php` as `secret`, and works together with `instanceid`.
If `instanceid` field is changed NextCloud will fail to decrypto files with `Bad Signature` error.

For more details see [the official documentation](https://docs.nextcloud.com/server/latest/admin_manual/configuration_files/encryption_details.html)

Other notable facts:

* Nextcloud generates a strong encryption key, which is unlocked by user’s passwords
* Encrypting files increases their size by roughly 35%
* Encryption keys are stored in `data/<user>/files_encryption` and `data/files_encryption`
* Encryption does not help in case of the server being compromised

# Tools

NextCloud provides a [set of `occ` commands](https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/occ_command.html#encryption) for managing the encryption setup:
```
 > docker exec -u 33 nextcloud-app php occ list encryption
Status 23.0.0

Usage:
  command [options] [arguments]

Options:
  -h, --help            Display this help message
  -q, --quiet           Do not output any message
  -V, --version         Display this application version
      --ansi            Force ANSI output
      --no-ansi         Disable ANSI output
  -n, --no-interaction  Do not ask any interactive question
      --no-warnings     Skip global warnings, show command output only
  -v|vv|vvv, --verbose  Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug

Available commands for the "encryption" namespace:
  encryption:change-key-storage-root     Change key storage root
  encryption:decrypt-all                 Disable server-side encryption and decrypt all files
  encryption:disable                     Disable encryption
  encryption:disable-master-key          Disable the master key and use per-user keys instead. Only available for fresh installations with no existing encrypted data! There is no way to enable it again.
  encryption:enable                      Enable encryption
  encryption:enable-master-key           Enable the master key. Only available for fresh installations with no existing encrypted data! There is also no way to disable it again.
  encryption:encrypt-all                 Encrypt all files for all users
  encryption:fix-encrypted-version       Fix the encrypted version if the encrypted file(s) are not downloadable.
  encryption:list-modules                List all available encryption modules
  encryption:migrate-key-storage-format  Migrate the format of the keystorage to a newer format
  encryption:recover-user                Recover user data in case of password lost. This only works if the user enabled the recovery key.
  encryption:scan:legacy-format          Scan the files for the legacy format
  encryption:set-default-module          Set the encryption default module
  encryption:show-key-storage-root       Show current key storage root
  encryption:status                      Lists the current status of encryption
```

There is also a separate [decryption tool](https://github.com/syseleven/nextcloud-tools/blob/master/rescue/decrypt-all-files.php) available.

# Links

* https://github.com/nextcloud/end_to_end_encryption_rfc/blob/master/RFC.md
* https://docs.nextcloud.com/server/latest/admin_manual/configuration_files/encryption_configuration.html
* https://docs.nextcloud.com/server/latest/admin_manual/configuration_files/encryption_details.html
* https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/occ_command.html#encryption
* https://github.com/syseleven/nextcloud-tools/blob/master/rescue/decrypt-all-files.php
* https://github.com/nextcloud/server/issues/8546
* https://github.com/status-im/infra-office/issues/7
