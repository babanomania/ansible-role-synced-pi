ansible-role-synced-pi
=========

This is an ansible-playbook to setup an raspberry-pi with periodic backups (with borgbackups) and online sync (with rclone)

- Weekly local backups of data via borgbackups
- Weekly online backups via rclone
- Ability to restore online backups via rclone
- discord alerts on backup and sync job completion

Requirements
------------


1. **Choose an online backup location**

    We can use rclone to keep our local backups online. It currently supports [over 40 cloud storage products](https://rclone.org/#providers). So if you need to keep your backups at dropbox use the guide [here](https://rclone.org/dropbox/) to create your rclone token.

2. **Create a discord server**

    We can create an discord server online easily, by using [this](https://support.discord.com/hc/en-us/articles/204849977-How-do-I-create-a-server-) url. Follow the guide [here](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks) to generate a webhook url. You'll be notified periodically on backups via discord


Role Variables
--------------

Below are the general parameters

| Variable Name | Description | Default Value |
|--|--|-- |
|data_restore|flag to restore data from online backups|False|
|base_path|base app for all setup|/app|
|backup_script|script to use to backup content to, before data is synced to online backups||
|restore_script|script to use to restore content from, after data is restored from online backups||



Below are the parameters related to [borgbackups](https://www.borgbackup.org)

| Variable Name | Description | Default Value |
|--|--|-- |
|borg_pass|borg password to encrypt your backups with|password|
|borg_repo_path|path to borg repository|/app/backup_sync/borgrepo|



Below are the parameters related to [rclone](https://rclone.org)

| Variable Name | Description | Default Value |
|--|--|-- |
|rclone_type|rclone remote type to sync backups to|dropbox|
|rclone_token|rclone token to use| { "access_token": "XXXX", "token_type": "Bearer", "refresh_token": "YYYY", "expiry": "ZZZZ" }|
|rclone_remote_path|rclone remote path|pi_backups|



Below are some additional parameters related to discord alerts

| Variable Name | Description | Default Value |
|--|--|-- |
|discord_alerts| flag to enable discord alerts| false |
|discord_webhook_url| discord webhook url | | 
|discord_botname|name of the discord bot|PiMaintenanceBot|


Example Playbook
----------------

```yaml
---
- hosts: raspberrypi

  vars:
    - data_restore: False

    - backup_script: |
        touch /app/backups/hosts
        cat /etc/hosts > /app/backups/hosts

    - restore_script: |
        touch /tmp/hosts
        cat /app/backups/hosts > /tmp/hosts

    # password used to encrypt the backups
    - borg_pass: password 

    # rclone configuration
    # Use this link to create the dropbox token https://rclone.org/dropbox/
    - rclone_type: dropbox
    - rclone_token: { "access_token": "XXXX", "token_type": "Bearer", "refresh_token": "YYYY", "expiry": "ZZZZ" }
    - rclone_remote_path: pi_backups

    - discord_alerts: False
    - discord_webhook_url: https://discordapp.com/api/webhooks/000/XXXYYY


  roles:
    - role: 'babanomania.synced_pi'
```

Tests
-----

Below are the steps to test this role

1. Copy the file __test/example.inventory.ini__ to __tests/inventory.ini__
2. Change the ip and username in the __tests/inventory.ini__ file to your raspberry pi's ip and default ssh user.
3. Ensure passwordless ssh is setup between your host and the raspberry pi
4. Run the below command

```bash
ansible-playbook -i tests/inventory tests/test.yml
```

License
-------

MIT
