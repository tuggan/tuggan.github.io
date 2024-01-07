---
title: "NextCloud rescue admin permissions"
date: 2024-01-07T17:50:59+01:00
hero: images/logos/nextcloud-hero.png
tags:
  - nextcloud
  - database
  - postgres
  - admin
---

While migrating my local users to oidc I accidentally removed admin privileges on the built-in admin user... This made continuing the migration difficult since i was unable to access the admin features of NextCloud. Here's a short description of how I rescued my privileges.

To give your user admin privileges you have to edit the NextCloud database. If you are running the NextCloud AIO you can access the database using the following command.

`sudo docker exec -it nextcloud-aio-database psql -U oc_nextcloud nextcloud_database`

There are three databases you have should know of

- oc_users\
    This table contains the local users.
- oc_groups\
    This table contains the groups.
- oc_group_user\
    And this table contains the groups a given user is a part of.

For example, to get a list of your users you could run the following command:

```SQL
nextcloud_database=> SELECT * FROM oc_users;
  uid  | displayname | password    | uid_lower 
-------+-------------+----------------+-----------
 admin |             | redacted ;) | admin
(1 row)

```

To add the local admin user to the local admin group you can run the following command:

```SQL
-- This starts a new transaction. It is not needed but I would recommend using
-- it since it will help you if you make a mistake.
BEGIN;

-- Inserts the values into the database.
INSERT INTO oc_group_user VALUES ('admin','admin');

-- This commits the transaction. Nothing will be changed until this command
-- is run. Verify that the previous commands have done what you expect before
-- running it. If there is something wrong you can run ROLLBACK; instead.
COMMIT;
```

You should just be able to reload the NextCloud interface and the admin interface will show up.

You can easily apply this to any user and group you have in your environment just change the first `admin` to the group and the 2nd to your user.
