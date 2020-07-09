---
title: Setting up permissions for apache var/www/html
layout: post
comments: true
published: false
description: 
keywords: apache, ubuntu
---

Using Apache with Ubuntu may cause permission issues in combination with PHP scripts and FTP clients. 
To address this issue, we need to set the correct user group and directory 
permissions for all sub-directories.

### Add your FTP user to the `www-data` group

```
usermod -a -G www-data username
```

... or create a new user:

```
sudo adduser username www-data
```

Replace `usename` with your FTP client username.

### Change the group and set the correct permissions

If your document root is `/var/www/html`, run these commands to fix 
the ownership and permissions for all files and directories.

```
sudo chgrp -R www-data /var/www/html/
sudo chown -R www-data: /var/www/html/
sudo chmod -R 2775 /var/www/html/
```

All FTP users have to login again for the changes to take effect.

### Read more

* <https://askubuntu.com/questions/244406/how-do-i-give-www-data-user-to-a-folder-in-my-home-folder>
* <https://gist.github.com/solancer/879105abb05eb7f13ce23e822ed70dd6>
* <https://askubuntu.com/questions/79565/how-to-add-existing-user-to-an-existing-group>
* <https://stackoverflow.com/questions/28918996/proper-owner-of-var-www-html>
