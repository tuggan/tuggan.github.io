---
title: "Migrate NextCloud AIO data directory"
date: 2024-01-07T16:03:24+01:00
hero: images/logos/nextcloud-hero.png
tags:
  - nextcloud
  - datamigration
---

When I faced the issue of migrating the storage directory for my AIO NextCloud the official README sent me to a github discussion whit a somewhat vague answer on how to perform the migration. I thought i would document the process for future reference and if this helps someone else perform a migration like this, that's a nice bonus.

# Pre migration tasks

Before performing any big changes like this you should backup everything. I did this in two ways.

1. I started by using the AIO interface to create a backup of NextCloud. **Remember to save the backup password**.
2. Then i created a snapshot of my NextCloud host so that i could perform a rollback on the entire host if needed. The NextCloud storage was not on this machine therefore it was not included in the snapshot.

# Migration

1. Log in to the AIO interface and stop all containers.
2. Stop the AIO master-container\
    `sudo docker stop nextcloud-aio-mastercontainer`
3. Take a snapshot of your host or perform a backup in some other way. This might include copying the docker volume directory (`/var/lib/docker/volumes/`) to another host.
4. Copy your files while preserving the permissions\
    `sudo rsync -aP  /path/to/NextCloudData /path/to/new/directory/`\
    This will create the directory NextCloudData under `directory`.
    - -a: Tells rsync to perform an archive copy. This preserves the important meta information like owner and permissions.
    - -P: Tells rsync to print progress.
5. Remove the master container\
    `sudo docker rm nextcloud-aio-mastercontainer`
6. Recreate the master container using your initial method but with the new directory set as the `NEXTCLOUD_DATADIR=/path/to/new/directory/NextCloudData`. I used my ansible role [docker-nextcloud-aio]().
7. Log in to AIO container and start all containers.
8. Log in to your NextCloud and make sure everything works.

# Post Migration

The data is now migrated to the new location and the master-container holds the information of where to find the data.
You might want to perform the same migration on your backup directory, though you should not save your user-data and backups in the same place.

# Conclusion

This migration was a lot easier than the documentation made it seem. I would be happy to see some official instructions in the documentation but i guess this is not something they want you to do since you could ruin your NextCloud setup.
