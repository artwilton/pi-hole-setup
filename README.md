# Pi-hole Setup Notes and Blocking Domains on a Schedule

Setting up my Raspberry Pi to run Pi-hole was fairly straightforward while following the beginner's guide linked below.

**The basic steps were:**
  - [Installing Rapsberry Pi OS](https://youtu.be/ntaXWS8Lk34), the Raspberry Pi Imager software makes this process very easy.
    - I installed Rapsberry Pi OS Lite, so I used [these instructions](https://www.stevencombs.com/raspberrypi/2016/05/18/change-terminal-font-size-raspi.html) to make the terminal font larger using `sudo dpkg-reconfigure console-setup`
  - Assigning a static IP address to the Raspberry Pi on my router.
  - Installing Pi-hole on the Raspberry Pi, and configuring it to run as a DHCP server as well.
    - On some Netgear routers, you might get the following error when trying to set the Raspberry Pi as your DNS Server:
      ```
      The IP address conflicts with the WAN IP subnet. Please eneter a different IP address
      ```
      To fix this you can simply keep your DNS settings at the default which is to let your ISP set your DNS, then reserve any static IP addresses for your Pi and disable DHCP on your router. After that you can point your DNS settings back to the Raspberry Pi. [This post](https://www.adamayala.com/blog/2019/08/06/netgears-static-ip-flaw-with-alternate-dns-servers/) from  Adam Ayala was helpful in solving that error.

## Helpful Links
- [Beginner's guide to building a Pi-hole](https://github.com/tgjohnst/pihole-guide)
- [Official Pi-hole Documentation](https://docs.pi-hole.net/)
- [Using pihole for time of day based per-client site blocking](https://thegreata.pe/articles/2021/02/28/pihole/)
- [Change terminal font size](https://www.stevencombs.com/raspberrypi/2016/05/18/change-terminal-font-size-raspi.html)

## Notes on setting up scheduled domain blocking
Using the helpful information on Thomas Mayfield's [blog post](https://thegreata.pe/articles/2021/02/28/pihole/), I created a custom Adlist for sites I wanted to block during certain times of the day.

Custom Adlist - https://raw.githubusercontent.com/artwilton/pi-hole-setup/main/social-media-blocklist.hosts

It's also possible to store an adlist file locally with the `file:///file-location` syntax. For example: `file:///home/pi/adlist.list`

**Shell Script to Enable and Disable custom Adlist:**
```
#!/bin/bash
#
#block_social_media.sh

sqlite3 /etc/pi/gravity.db "update adlist set enabled = $1 where id = 2;"
pihole restartdns reload-lists
```

- **Troubleshooting Note:** After trying a few different combinations of commands, I eventually found that `pihole restartdns reload-lists` worked best for consistently enabling and disabling the adlist, without having to restart any services like the `pihole restartdns` command by itself does.

  `pihole -g` also seemed to work well but I didn't want to have to rebuild the gravity database outside of its normal schedule, and when looking at the [code for enabling and disabling adlists in the Web interface](https://github.com/pi-hole/AdminLTE/blob/64bbce906b31c213183db7b6f0547513423e4bbf/scripts/pi-hole/php/groups.php#L1232) I saw that the Pi-hole devs use `pihole restartdns reload-lists` as well.

  What I believe also helped was making sure to include `bash -lc` when running `sudo crontab -e`, which was the syntax used in Thomas Mayfield's blog post above. According to [this post](https://unix.stackexchange.com/a/121489), this forces "cron execution to use your login environment."

**Example syntax for crontab** which passes a "true" or "false" parameter to the shell script. This would un-block social media websites from 8:00 AM to 11:00 AM.
```
# m h dom mon dow command
00 8 * * * bash -lc "/home/pi/block_social_media.sh false"
0 11 * * * bash -lc "/home/pihole/block_social_media.sh true"
```
