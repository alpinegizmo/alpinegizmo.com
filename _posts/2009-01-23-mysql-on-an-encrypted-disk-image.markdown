---
layout: post
title: Mysql on an encrypted disk image
---

Yesterday I decided it would be a good idea to modify the installation of mysql that I use on my laptop so that the data on the disk is encrypted. With google I found a few people looking for help with this, but no solutions, so I'll share what I figured out.

First some configuration details. I'm running mysql 5.0 on a macbook pro with leopard (OS X 10.5.6). I'm pretty sure the same approach I'm using will work across any similar setup, but your mileage may vary. Also, I installed mysql from mysql.com, but you should be able to easily adapt this technique to work with the version from macports.

After shutting down the mysql server, the first thing I did was to create an encrypted sparse disk image (using disk utility), move /usr/local/mysql/data to the new disk image, and create a symbolic link from /usr/local/mysql/data to the new, encrypted location. I expected that to pretty much take care of it (and if it had, I wouldn't be blogging about it). When I looked at why mysql would no longer start up, I found that the wrong user now owned the data files -- I owned everything, instead of the mysql user. Nothing chown can't fix, right? Wrong.

Turns out that the files and directories in mounted disk images are owned by whoever mounts them -- the ownership information stored in the filesystem within the disk image is ignored. Yikes! Fortunately, this is merely the default behavior, and can be switched off. If you bring up a mounted disk image in the finder's get info window, at the very bottom, inside the "Sharing & Permissions" section, you will find a checkbox labeled "Ignore ownership on this volume". It is checked by default; you want to turn this off.

<img src="/images/ignore-ownership.png">

You will notice that I gave everyone read and write permission to this volume. I'm not super happy about this, but so far I haven't found any other way to allow the mysql user to be able to reach inside and do what it needs to do. Of course, what this volume contains is a data directory that only the mysql user is able to access, so I don't feel too bad. Nevertheless, if someone finds a solution to this, let me know, ok?

The other problem I've found comes up when I want to unmount the mysql data disk image. I usually find the finder telling me I can't eject this volume because it is still in use. In these cases lsof is my friend, as in "lsof /Volumes/mysql-data". And the culprit is usually mds, aka spotlight. You can stop this by going into System Preferences > Spotlight > Privacy and adding the encrypted disk image volume to the list of things spotlight should not index. Unfortunately it seems that this preference is not sufficiently persistent -- I keep having to re-set it -- so I may resort to disabling spotlight entirely, or try [spotless](http://www.fixamac.net/software/spot2/index.php).

Of course, it is also necessary to mount the encrypted disk image before starting mysql, and to stop mysql before unmounting it. Kudos to anyone who goes to the trouble of cleanly automating these steps.
